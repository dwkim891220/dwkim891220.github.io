---
title: "Android Java에서 Kotlin으로"
categories: 
- Develop
tags:
- Kotlin
last_modified_at: 2018-01-22
toc: true
---

처음 웹 어플리케이션 개발을 하면서 배운 java를 지금까지 사용하고 있다.  

[Xamarin](https://visualstudio.microsoft.com/xamarin/) 을 사용하기 위해 6개월 정도 `C#` 을 사용했던 기간을 제외하곤 모두 `java`를 사용 했다.  

`Android Studio`에서 `java1.7`을 사용 하면서  `C#` 에서 유용하게 사용했던 `LinQ`, `lambda` , `delegate`와 같은 좀 더 이용자 편의에 맞춰진 기능들이 아쉬웠다.

어쩔 수 없이 `java`를 사용해 개발을 하던 중 안드로이드 프로젝트에 `kotlin`을 도입한 글을 보게 되었다.  

언젠가부터 구글 예제도 `kotlin`으로 작성이 되기 시작하고, `kotlin`도입에 관심을 가지기 시작했다.

그러던 중 왜 `kotlin`을 도입해야 하는지 이유는 분명했다.  



### ___- 왜 `kotlin` 인가?___

- `AndroidStudio`가 새로운 버전이 나오면서 `java1.8`을 지원하기 시작했다. 하지만 minSdk가 15인 현재 프로젝트에 적용 할 수 없었다. `java1.8`을 사용하기 위해선 minSdk를 15 이상으로 변경해야 하는 상황이었다.

- `Nullable` 타입이 아님에 따른 `NullPointerException`처리

  ```java
  if(foo != null){
      foo.something();
  }
  
  try{
      foo.something();
  }catch(NullPointerException exception){
      //something
  }
  ```

- 안드로이드 보일러 플레이트 코드들

  ```java
  button.setOnClickListener(new View.OnClickListener{
      @Override
      void onClick(View view){
          //something
      }
  });
  ```

- `ButterKnife` 보일러 플레이트 코드

  ```java
  @BindView(R.id.btn_main)
  TextView btn1;
  
  btn1.setText("test");
  ```

- `Collection`함수들의 한계  

  ```java
  ArrayList orgList;
  ArrayList retList;
  
  for(Item item : orgList){
      item.something();
      retList.add(item);
  }
  ```


### ___- 어떻게 개선 했나?___

- `NullPointerException` 대응

  ```kotlin
  foo?.something()
  ```

- 안드로이드 보일러 플레이트 코드 대응

  ```kotlin
  button.setOnClickListener{
      //something()
  }
  ```

- ButterKnife 제거 후 [kotlin/Anko](https://github.com/Kotlin/anko) 로 대체

- ```kotlin
  btn1.text = "test"
  ```

- `Collections`

- ```kotlin
  val orList = arrayListOf()
  val retList = orgList.map{ item ->
      item.something()
  }
  ```



### ___- 마무리___

보일러플레이트 코드 감소와, 기본 제공하는 유용한 기능들 (`lambda`, `lateinit`, `lazy`  등등)로 인해 개발 속도 및 확장성은 무수히 많다고 느꼈습니다.  

`java`와 `kotlin`을 함께 사용하면서 호환성 문제로 몇몇 처리가 필요했지만, 이 문제들은 빠르게 개선 되었고 `kotlin`을 100 % 사용함에 따라 문제는 사라졌습니다.  

2017~2018년 `Android`관련 주제중 가장 핫한 키워드는 `kotlin`으로 생각 됩니다.  

가장 핫 한 만큼 문제에 대한 대처가 빠르게 진행 되고, 무수히 많은 이용자들이 검증을 해 주었고, 그 덕분에 좀 더 쉽게 `java`에서 `kotlin`으로 옮겨 가지 않았나 생각 됩니다.  

새로운 언어를 제품에 적용하는 작업은 매우 즐거운 경험으로 남았습니다.