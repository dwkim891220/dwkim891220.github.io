---
title: "Volley 에서 Retrofit2 로 변경한 이야기"
categories: 
  - Develop
tags:
  - Retrofit2
  - Volley
last_modified_at: 2017-07-27
toc: true
---

새로운 회사로 이직 후 기존에 서비스 중이던 안드로이드 어플리케이션을 담당하여 개발 및 유지보수를 하게 되었다. 이전부터 네트워크 라이브러리는 Retrofit2 를 사용했었고 이번에 처음으로 Volley로 구현 된 코드를 보게되었다. 앱 수정 요청이 있어 수정을 하던 중 몇가지 문제점에 부딪히게 된다.

### ___- 서버에 Get Request 하면서 parameter의 Url Encoding 문제___

- 문제 발생

보유중인 테스트폰 2대(Samsung Galaxy S6 (7.0) / LG G4 (5.0))로 구현한 기능을 테스트 중 동일한 Request 코드임에도 각각 휴대폰에서 다른 결과를 보임.

동일한 Request에 동일한 파라미터를 추가해 요청하는데 LG G4에서만 crash 발생



- 문제 파악

exception 내용은 `E/Volley: [3417] BasicNetwork.performRequest: Unexpected response code 400 for` 

response 의 htmlStatusCode가 400이다. 서버개발자의 도움을 받아 서버와 함께 디버깅을 시작한다. 해당 api를 디버깅 해보니 get Request에 parameter가 포함되지 않았다. 즉 'StringRequest'에 parameter가 안 붙고 그대로 request 되고 있었다. 하지만, 하나의 기기에서는 정상적으로 Parameter가 포함되고, Request 생성시 Header에 Content-Type이 설정되지 않아 기본 encoding이 다를수도 있겠다는 생각에 Header를 살펴보니 이미 Content-Type이 UTF-8로 설정되어 있다.



- 문제 해결

코드에서 문제가 발생하지 않았을거란 전제하에 기기의 차이에서 문제를 찾아본다. 추가적으로 다른 안드로이드 기기에서 테스트해보았다. Samsung Galaxy Note4 (6.0) 에서도 LG G4 (5.0)과 동일하게 문제를 발생시킨다. 추가적으로 LG V10 (7.0)에서 테스트했더니 정상 작동한다. 원인의 범위가 좁혀진다. Android OS 버전 7.0 과 6.0을 경계로 동일한 코드가 다르게 작동한다. 문제 해결을 위해 코드에 URL을 강제로 UTF-8로 인코딩 해보았다.

```java
try{
    keyword = URLEncoder.encode(keyword, "UTF-8");
} catch (UnsupportedEncodingException e){
 	e.printStacktrace();
}

Log.e(class.getName(), ApiUrl.API_APP_URL + ApiUrl.API_KEYWORDS + "?q=" + keyword)
    
HttpProxy.getInstance(context).addStringRequest(
    ApiUrl.API_APP_URL + ApiUrl.API_KEYWORDS + "?q=" + keyword,
    HttpProxy.GET,
    null,
    new HttpProxy.HttpProxyListener() {
        @Override
        public void onReady(){
            
        }
    }
);
```

keyword를 강제로 인코딩 하는 코드를 추가하였다 `URLEncoder.encode( string , “UTF-8”);`.

테스트해보니 모든 기기에서 정상 작동한다. 너무나도 허무했다. 좀 더 깊숙히 문제를 파악하고 싶었지만, 시간이 부족하여 해결한 내용을 정리하고 넘어가기로 한다. Volley 에서 StringRequest 생성시 URL 을 직접 String 으로 생성시 Encoding 직접 해줘야 한다.

<https://stackoverflow.com/questions/11212173/different-charset-in-android-os-version>

이 질문에도 정확한 답변은 찾을 수 없었지만, 비슷한 문제를 겪는사람이 있다는것에 위안을 삼으며 문제를 마무리 지었다.



___- ErrorResponse 에 ResponseBody가 없는 문제___

- 문제 발생

서버와 Api 프로토콜을 정의하는데 있어 서버를 기준으로 클라이언트도 정의된다. Api 통신하는데 Response의 형식 및 Type을 예측하고 결과를 처리하는 코드들이 수없이 존재한다. 400 badRequest 같은 status code에 따라 처리를 해줘야 하는 경우가 생겼다. status code가 400일 경우 Response 의 메시지를 화면에 보여줘야 한다. Volley에서 StringRequest를 생성하면서 정상 결과를 가져오는 `onResponse` 와 비정상 결과를 가져오는 `onErrorResponse` 를 등록해준다. status code가 200일때 `onResponse` 로 외의 코드일때 `onErrorResponse` 로 각각 결과를 리턴해준다. 이제 400 일 경우 처리를 하기위해 리스너를 확인한다.

```java
@Override
public void onErrorResponse(VolleyError volleyError) {
    System.out.println(volleyError.getMessage());
}
```



- 문제 파악

volleyError 인스턴스에 값을 디버거를 통해 확인해보았지만, return Value에 대한 body 에 포함 된 어떤 데이터도 찾을수가 없었다.

```java
@Override
public void onErrorResponse(VolleyError error) {
    String body;
    //get status code here
    String statusCode = String.valueOf(error.networkResponse.statusCode);
    //get response body and parse with appropriate encoding
    if(error.networkResponse.data!=null) {
        try {
            body = new String(error.networkResponse.data,"UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
    //do stuff with the body...
}
```

구글 결과 위의 코드로 body를 가져올 수 있다고 한다. 바로 적용해본 결과 

![01]({{ "/assets/images/post/2017-07-27-migrate_Volley_to_Retrofit2_story/01.png" | absolute_url }})

디버거를 확인 해 본 결과 `error.networkResponse.data` 가 비어있다. 많은 StackOverFlow 를 참조하고 방법을 찾아보았지만, 문제를 해결할 수 없었다. 이 문제를 계기로 통신 라이브러리 변경을 고민하게 되었다.



#### ___마무리___

위의 두가지 문제중 ErrorResponse 에 ResponseBody가 없는 문제는 치명적이었습니다. 사용하는 대부분의 Api Response가 errorCode와 함께 contents를 갖고있기 때문. 또한 이미지 업로드를 하기위해 MultiParts Request도 필요했는데, 이 기능을 위해 코드 추가 및 수정이 필요했고, 위 두가지 치명적인 문제를 계기로 라이브러리 변경을 결심했습니다. 또한 [HttpClient 가 deprecate 되어](https://developer.android.com/about/versions/marshmallow/android-6.0-changes?hl=ko#behavior-apache-http-client) 이 후 volley의 지원 중단도 라이브러리 변경하게 된 이유 중 하나 였습니다. 이 후 대체 라이브러리를 선택 하는데 있어, 익숙하고 많은 사용자가 사용하는 Retrofit2로 선택하게 되었고, 위 두가지 문제를 해결 가능한지 구글링을 통해 확인 후 작업을 시작하였습니다. 위 문제들을 Retrofit2로 변경하면서 어떻게 해결했는지 다음 글에서 포스트 하도록 하겠습니다.