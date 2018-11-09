---
title: "Android + Kotlin 편리하게 느낀 기능들(2)"
categories: 
- Develop
tags:
- Kotlin
last_modified_at: 2018-03-22
toc: true
sitemap :
  changefreq : daily
  priority : 1.0

---

이전 포스트에서 간략하게 `null check`,`lateinit`,`let, run, apply` 에 대해 알아봤습니다.  

이번 포스트에서는 `kotlin extension`과 `kotlin default argument`, `data class` 등을 알아보도록 하겠습니다.  

### - Kotlin Extention

확장 함수는 기존에 정의 되어 있는 객체나 함수를 사용자 확장 정의해서 사용할 수 있도록 지원하는 기능.  

기존 자바에서 이와 같은 기능을 하려면 상속가능한 클래스를 상속받아 구현해야 하는 번거로움이 있지만, 코틀린에서는 어느 위치에서든 단순 선언으로 쉽게 사용 가능하다.

```kotlin
fun View.show(show: Boolean = true){
    this.visibility = if(show) View.VISIBLE else View.GONE
}

fun View.isVisibility() : Boolean{
    return this.visibility == View.VISIBLE
}
```

위 처럼 이미 존재하는 `View` class에 show라는 메소드를 선언해 사용을 할 수 있다.  

위 코드에서 this는 `View` class로 접근 가능하다.

### - Default argument

`java`는 기본적으로  `overLoading`을 지원한다. `overrLoading`으로 동일한 이름의 인자만 다른 메소드들을 여러개 선언할 수 있지만, 그에 따른 보일러플레이트 코드들도 반복 된다.  

하지만 인자 기본값을 설정 할 수 있어 아래 처럼 사용 가능하다.

```kotlin
fun addItem(id: Int = 0, name: String = "defaultName", age: Int = 1){
    Log.d("test","id = ${id}, name = ${name}, age = ${age}")
}

addItem() //id = 0, name = defaultName, age = 1
addItem(id = 10) // id = 10, name = defaultName, age = 1
addItem(id = 1, name = "test", age = 11) // id = 1, name = test, age = 11
```

기본 인자를 설정함에 따라서, 어떤 인자에 어떤 값을 줄지도 강제로 설정할 수 있다.  

`java`에서는 `overrLoading`된 메소드들에 어떤 인자가 어떻게 들어가는지 눈으로 확인 가능해 실수도 줄일 수 있다.

또한 인자명을 강제로 넣어, 인자의 순서도 바꿀 수 있다.

### - Data class

통신 라이브러리인 `retrofit`을 사용하다 보면, 결과를 `class`로 선언해서 지정하면 자동으로 형 변환을 해준다(GSON)  

결과 `class`에는 멤버 타입과 변수명 등 서버와 협의 된 내용들로 채워야 하는데, getter, setter 또한 함께 선언이 되어야 한다.  

코틀린에서는 `data class`를 지원하는데 `java`의 getter setter를 자동으로 만들어준다.

```java
public class Person(){
    String name;
    int age;
    
    String getName(){
        return name;
    }
    
    int getAge(){
        return age;
    }
    
    void setName(String pName){
        this.name = pName;
    }
    
    void setAge(int pAge){
        this.age = pAge
    }
}
```

위와 같은  `class`를 `kotlin` 에서는

```kotlin
data class Person(
	val name: String,
    var age: Int
)
```

로 간단하게 선언 가능하다.  

실제로 위의 `data` 키워드만 붙이면, 보이지는 않지만 자동으로 setter, getter가 선언 되어 getName(), setName("name") 과 같이 사용이 가능하다.  

필요에 따라 읽기전용으로 setter가 필요 없다면, `name` 멤버처럼 `val`로 선언하면 setter는 사용할 수 없고, getter만 사용 가능하다.  

### - 마무리

지금까지 소개한 기능들은 극히 일부분이며, `kotlin`이 제공하는 수많은 기능들을 사용하다 보면, 정말 편리하게 개발 할 수 있습니다.  저 또한 모든 기능을 알지 못하고 활용할 수 없지만, 소개한 기능들로도 `java`를 사용할때 보다 충분히 만족 스럽습니다.