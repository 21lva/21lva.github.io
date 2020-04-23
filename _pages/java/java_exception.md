---
layout: post
title:  "예외 처리"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_exception.md
---

프로그램 오류
============

1. 컴파일 에러 : 컴파일 시에 발생하는 에러
2. 런타임 에러 : 실행 시에 발생하는 에러
  - 에러(error) : 프로그램 코드에서 수습될 수 없은 오류. ex) StackOverflowError, OutOfMemoryError
  - 예외(exception) : 프로그램 코드에서 수습될 수 있는 오류. ex) NullPointerException
3. 논리적 에러 : 실행은 되지만, 의도와 다르게 동작하는 것

- - -

계층 구조
==========

- 자바에서는 실행 시 발생할 수 있는 오류를 클래스로 정의

![예외 계층도]({{site.baseurl}}/post_img/{{page.name}}/exception_hierachy)

- **Exception** 클래스는 크게 **RuntimeException** 와 아닌 것들로 나뉜다
- **RuntimeException** : 프로그래머의 실수에 의해서 발생할 수 있는 예외들
- **Exception** : 외부 영향으로 발생할 수 있는 오류. 사용자의 실수 등

- - -

예외 처리하기
============

- 예외 처리 : 프로그램 실행시 발생할 수 있는 예외에 대비한 코드를 작성. 비정상적 종료를 막고 정상적인 실행을 보장. **try-catch** 사용

```java
try{
  //예외가 발생할 수 있는 코드
}catch(Exception1 e1){
  //Exception1에 대한 처리 코드
}catch(Exception2 e2){
  //Exception2에 대한 처리 코드
}catch(Exception3 e3| Exception4 e4){
  //Exception3과 Exception4에 대한 처리 코드
}finally{
  //예외 발생 여부와 상관없이 실행되어야 하는 코드
}
```

- - -

예외 발생시키기
=============

throw
------

- 예외를 발생시키려면 다음의 과정을 거친다
  1. new를 이용하여 해당하는 예외 클래스를 생성한다
  2. **throw** 를 통해서 예외를 발생시킨다
- 물론, 1과 2를 동시에 할 수도 있다

```java
public class Test {
    public static void divide(int a,int b) {
        if(b==0) {
            throw new ArithmeticException();
        }
    }

    public static void main(String[] args) {
      func(1,2);
      func(2,0);
    }
}
```

- 자기만의 예외 클래스를 생성하고자 하면 **Exception** 클래스를 상속 받는다

```java
class myException extends Exception{

}
```

throws
------

- 메서드에서 발생하는 예외를 다루려면 선언부에 **throws** 를 이용
- 보통 **RuntimeException** 이 아닌 예외들을 선언한다.
- 예외를 위로 메서드 내부에서 처리하지 않고 상위 메서드에서 처리하도록 미루는 방법
- 상위로 올라가다가 **main** 에서도 예외를 처리하지 않으면 프로그램이 종료됨
- 상속후 오버라이딩할 때에는 부모 메서드의 예외보다 많은 예외를 적용할 수 없다


```java
public static void fun(int a) throws Exception1, Exception2{
  //메서드 구현부
}
public static void main(String[] args){  
  try{
    fun(11);//1번 fun
    fun(12);//2번 fun
  }catch(Exception1 e1|Exception2 e2){
    //예외 처리  
  }
}
```

- 상위 메서드인 main에서 예외를 처리하면 1번 함수에서 예외가 발생한 경우에는 2번은 동작하지 않는다.
- fun메서드에서 예외를 처리하게 하면 1번에서 오류가 나도 2번이 진행된다

예외 되던지기(re-throwing)
------------------------

- 일부 예외는 메서드 자체에서 처리하고, 일부는 그 메서드를 호출한 메서드에서 처리하도록 하는 방법

```java
static void method() throws Exception{
  try{
    //코드
  }catch(Exception e){
    //일부 처리
    throw e;
  }
}
```
<!--
연결된 예외 처리
-->
