---
title: "Retrofit2 에서 CustomCallback 구성"
categories: 
- Develop
tags:
- Retrofit2
last_modified_at: 2017-10-17
toc: true
sitemap :
  changefreq : daily
  priority : 1.0
---

Volley 에서 Retrofit2로 변경 후 http request 에 대한 response의 status code에 따른 통합 처리가 필요했다.  

status code가 400일때 에러 dialog를 보여주고, 401일때는 로그인 화면으로 이동해야 했다.  

- 현재 방식

```java
call.enqueue(new Callback<DevicesResult>() {
    @Override
    public void onResponse(Call<Result> call, Response<Result> response) {
        int code = response.code();
        DevicesResult body = response.body();
        switch (code){
            case 200:
                // something
                break;
        }
    })
```

각각의 request에 따른 response를 일일히 변환하고, code에 대한 구분을 하고, 처리 하도록 하였다.  

현재 방식처럼 처리를 할 경우, 예를 들어 서버에서 특정 status code를 변경하여 client의 모든 code들을 변환하게 되었을때 충분히 문제가 발생할 수 있는 구조였다.

위와 같은 문제를 해결하기 위해, `Callback<T>`을 상속받아 새로운 `CustomCallback<T>`를 만들어 사용하기로 하였다.

- 새로운 방식

```java
public class CustomCallback<T> implements retrofit2.Callback<T> {
    final int SUCCESS               = 200;

    final int INVALID_PARAMETER     = 400;
    final int NEED_LOGIN            = 401;
    final int UNAUTHORIZED          = 403;
    final int NOT_FOUND             = 404;

    final int INTERNAL_SERVER_ERROR = 500;
    
    public interface ResponseListener<T>{
        void successResponse(T response);
        void errorResponse(ResponseBody response);
    }

    Context mContext;
    NetworkLoadingDialog mLoadingDialog;
    ResponseListener mListener;

    public CustomCallback(Context context,
                          NetworkLoadingDialog loadingDialog,
                          ResponseListener listener) {
        mContext = context;
        mLoadingDialog = loadingDialog;
        mListener = listener;
    }

    @Override
    public void onResponse(retrofit2.Call<T> call, retrofit2.Response<T> response) {
        if(mLoadingDialog != null) mLoadingDialog.dismiss();

        int code = response.code();
        T body = response.body();
        ResponseBody errorBody = response.errorBody();

        switch (code){
            case SUCCESS:
                if(mListener != null) mListener.successResponse(body);
                break;
            case INVALID_PARAMETER:
            case INTERNAL_SERVER_ERROR:
            case NOT_FOUND:
                if(mListener != null) mListener.errorResponse(errorBody);
                break;
            case UNAUTHORIZED:
            case NEED_LOGIN:
                loginProcess();
                break;
            default:
                Log.d("CustomCallback", "error code = "+code);
                break;
        }
    }

    @Override
    public void onFailure(retrofit2.Call<T> call, Throwable t) {
        t.printStackTrace();
        if(mLoadingDialog != null) mLoadingDialog.dismiss();
    }

    private void loginProcess(){
        // clear login
    }
}
```

위와 같은 CustomCallback 클래스를 만들어 모든 request 에 적용 하였다.

```java
call.enqueue(new CustomCallback<Result>() {
    @Override
    public void onResponse(Call<Result> call, Response<Result> response) {
        // something
    })
```

api 호출에 대한 response에서 서버의 status에 대한 처리가 사라져, 하나의 클래스에서 일괄 된 처리가 가능하다.  

`retrofit2.Callback<T>.errorResponse` 처럼 통신에서 발생하는 예외가 아닌, 서버와 협의 된 특정 코드에서도 일괄 처리가 가능해졌다.  



### ___- 마무리___

Volley 에서 Retrofit2으로 어찌 보면 무리한 변경을 하였지만, 사용하면서 Retrofit2의 심플한 구조와 확장성으로 인해 매우 만족스럽게 사용하고 있습니다. [Retrofit2로 변경하게 된 이유]({{ site.url }}{{ site.baseurl }}/develop/migrate-Volley-to-Retrofit2-story/)