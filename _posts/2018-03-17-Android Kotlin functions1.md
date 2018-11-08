---
title: "Android + Kotlin 편리하게 느낀 기능들(1)"
categories: 
- Develop
tags:
- Kotlin
last_modified_at: 2018-03-17
toc: true
sitemap :
  changefreq : daily
  priority : 1.0
---

`kotlin`을 제품에 적용하면서 너무나도 쾌적한 개발 환경을 느꼈습니다.  

어떤 부분에서, 어떻게 느꼈는지 조금씩 정리 해 보도록 하겠습니다.  

### - Null check

```java
if(foo != null){
    foo.something();
}else{
    otherSomething();
}
```

위와 같이 `NullPointerException`을 막기 위해 코드를 작성 해야 함.

```kotlin
foo?.run{
    something()
} :? otherSomething()
```

불필요한 `if else`가 사라지고, 간단하게 작성 가능.  

하지만 method chain으로 접근 할 경우 부득이 하게 모든 메소드 뒤에 `?`를 붙여줘야 한다.

```kotlin
val latLng = googleMap?.projection?.visibleRegion?.latLngBounds
```

아래와 같은 케이스는 사용 해보진 않았지만 충분히 가능.

```kotlin
googleMap?.run{
    projection?.run{
        visibleRegion?.run{
            latLngBounds
        } ?: something3()
    } ?: something2()
} :? something1()
```

보기만 해도 끔찍하지만... 반복되는 수많은 `if else` 구문에 비하면 양호하다고 생각 됨.

### - lateinit 

```java
interface FooListener{
    void onClickFoo();
}

class A {
    FooListener mListener;
    
    public A(FooListener listener){
        mListener = listener;
    }
    
    void onClick(){
        if(mListener != null){
            mListener.onClickFoo();
        }
    }
}

new A(new FooListener{
    @Override
    void onClickFoo(){
        //something
    }
})
```

위와 같은 listener pattern을 많이 사용하게 되는데, 아래 처럼 사용할 수 있음.

```kotlin
class A {
    lateinit var onClickFoo: () -> Unit
    
    void onClick(){
        onClickFoo()
    }
}

A().apply{
    onClickFoo = {
        //something()
    }
}
```

불필요한 interface 선언이 필요 없고, 보일러 플레이트 코드들도 많이 줄어 들었다.  

하지만 위 처럼 사용하게 될 경우 onClickFoo 에 대한 정의를 하지 않게 되면 `UninitializedPropertyAccessException` 가 발생한다.  

```kotlin
if(::onClickFoo.isInitialized) onClickFoo()
```

위 코드로 `UninitializedPropertyAccessException`를 예방할 수 있다.  

### - run, let, apply

- run 

  run은 return 값이 없다. 그러므로 이미 생성 된 인스턴스에 대한 접근에 적절하다. return 값이 필요 없고, 반복적으로 인스턴스에 접근 해야 하는 케이스에 적합하다.

  ```kotlin
  //변경 전
  tv_main_name.text = "텍스트"
  tv_main_name.visibility = View.VISIBLE
  
  //변경 후
  // this = tv_main_name
  tv_main_name.run{
      this.text = "텍스트"
      this.visibility = View.VISIBLE
  }
  
  // this 생략 가능
  tv_main_name.run{
      text = "텍스트"
      visibility = View.VISIBLE
  }
  ```

- let

  let은 run과 비슷하지만 2가지 차이점이 있다. return  값을 가지고, 람다식의 인자 이름을 가질 수 있다.

  ```kotlin
  //변경 전
  tv_main_name.View.VISIBLE
  val nameText: String = tv_main_name.text.toString()
  
  //변경 후
  val nameText: String = tv_main_name.let{ tv ->
      tv.visibility = View.VISIBLE
      tv.text.toString() // return 값
  }
  
  // 인자에 대한 구분이 필요 없고, 인자가 1개 인 경우
  // tv 라는 인자 대신 it 으로 접근 가능
  val nameText: String = tv_main_name.let{
      it.visibility = View.VISIBLE
      it.text.toString() // return 값
  }
  ```

하지만 위와 같은 상황에서 let을 사용하는 경우는 드물다.  

run 은 인자에 대한 이름을 지정할 수 없어 구문 안에서 수많은 인스턴스에 접근 할 경우에 가독성이 떨어진다.  

이럴 때 let을 사용해 인자에 대한 이름을 지정해 코드에 대한 구분을 높일 수 있다.

```kotlin
tv_main_name.run{
    this@MainActivity.something()
    this.visibility = View.VISIBLE
    this@MainActivity.something()
}

tv_main_name.let{ tv ->
    something()
    tv.visibility = View.VISIBLE
    something()
}
```

- apply

  apply 는 run 과 비슷하지만, 객체 자신을 return 한다.

  ```kotlin
  //변경 전
  val tv = TextView(context)
  tv.text = "Text"
  tv.visibility = View.GONE
  val mainTv: TextView = tv
  
  //변경 후 
  val mainTv = TextView(context).apply{
      text = "Text"
      visibility = View.GONE
  }
  ```



위 3가지 함수는 매우 자주 사용하지만, 막상 설명하려 하면 너무나도 헷갈려 매번 설명글을 참조하게 됩니다.  

그 중 한국어로 아주 잘 설명 해 주신 [커니의 안드로이드](https://www.androidhuman.com/lecture/kotlin/2016/07/06/kotlin_let_apply_run_with/)를 참조하시면 좀 더 많은 정보를 얻을 수 있습니다.  

위 3가지 함수만 적절히 사용해도 코드의 양은 줄어들고, 가독성은 늘어나게 됩니다. 매우 자주 사용하고, 유용한 함수들이니 직접 사용 해 보고 적절히 사용하면 큰 도움이 될 것 입니다.