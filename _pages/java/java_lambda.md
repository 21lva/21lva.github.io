---
layout: post
title:  "Lambda expression"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_lambda.md
---

람다식(lambda expression)
======

- 메서드를 하나의 **식(expression)** 으로 표현한 것
- 메서드의 이름이 없으므로 **익명함수(anonymous function)** 이라고도 함

- **(매개변수)->{
  문장
}**

- return이 있으면 expression으로 대체(**;** 붙이지 않는다)
- 매개변수가 추론 가능한 경우(대부분)은 생략 가능
- 매개변수가 하나인면 **()**  생략 가능
- 식이 한줄이면 **{}** 생략 가능. BUT return문이면 생략 불가능

```java
//둘 중 큰 값 반환하는 람다식
(a,b) -> a>b?a:b

//매개변수가 하나이면 괄호 생략 가능
a -> a*a
```

- 람다식은 익명 클래스의 객체와 동등
- 변수처럼 메서드를 통해서 람다식을 주고 받을 수 있음
- Functional Interface : 람다식을 하나의 인터페이스로 다루는 것
- 함수형 interface에는 오직 하나의 추상 메서드만 정의되어야 함. BUT static메서드, default 메서드는 제약 없음

```java
@FunctionalInterface
interface Myfunc{
  public abstract int max(int a,int b);
}
```

- 다음과 같인 메서드를 쉽게 구현할 수 있다

```java
//람다식을 쓰지 않는 경우
Collection.sort(list,new Comparator<String>(){
  public int compare(String s1,String s2){
    return s2.compareTo(s1);
  }
});
  
//람다식을 사용
Collection.sort(list,(s1,s2)->{
  s2.compareTo(s1)
});
```

- 람다식은 컴파일러가 임의로 이름을 지정하기 때문에 타입을 알 수 없음
- 대입 연산자를 이용하려면 형변환을 해야함
- FunctionalInterface 의 경우에만 형변환이 가능

```java
MyFunc f = (MyFunc)(()->());

//불가능
Object f = (Object)(()->());
```

외부 변수 접근
-------------

- 람다식에서 외부 지역 변수에 접근하면 그 변수는 final이 붙지 않아도 상수로 간주
- BUT 멤버변수는 final로 생각하지 않음
- 외부 지역변수와 람다식 매개변수의 이름은 달라야 함
```java
class A{
  int val = 30;
  void method(int i){
    final int val = 12;
    Myfunc f = ()->{
      System.out.println("i : "+i);
      System.out.println("val : "+.val);
      System.out.println("this.val : "+ ++this.val);
    }
  }
}
```

메서드 참조
-----------

- 람다식을 더 간단하게 사용하는 방법
- 하나의 메서드만 호출하는 경우에 사용

```java
//원래 람다식
Function<String,Integer> f= (String s) -> Integer.parseInt(s);
BiFunction<String,String,Boolean> f= (s1,s2) -> s1.equals(s2);

//메서드 참조. 생략된 부분은 컴파일러가 알아서 추측
Function<String,Integer> f = Integer::parseInt;
BiFunction<String,String,Boolean> f= String::equals;//equals를 String의 equals로 특정지어줘야 함

//인스턴스 메서드 참조
MyClass obj = new MyClass();
Function<String,Boolean> f = obj::equals;

//생성자 호출
Supplier<MyClass> s = MyClass::new;

//매개변수 있는 생성자. 알맞은 함수형 인터페이스를 골라야 함
Function<Integer,MyClass> f = MyClass::new;


//배열 생성
Function<Integer,int[]> f = x ->int[]::new;
```

- - -

함수형 인터페이스
===============

- java.util.function 패키지 : 일반적으로 자주 쓰이는 형식의 메서드를 함수형 인터페이스로 정의. 매번 새로운 인터페이스를 만들지 말고 이 인터페이스를 이용한다
- 함수형 인터페이스 : 오직 하나의 추상메서드만을 가지는 인터페이스

- Supplier<T> : 매개변수는 없고, 반환값만 있음
- Consumer<T> : 매개변수만 있고, 반환값 없음
- Function<T,R> : 매개변수 1개 반환값 1개
- Predicate<T> : 조건식에 사용. 매개변수1개와 반환타입 boolean

- BiConsumer<T,U> : 매개변수 2개, 반환값 없음
- BiPreidcate<T,U> : 조건식. 매개변수 2개와 boolean 반환
- BiFunction<T,U,R> : 2개의 매개변수 + 하나의 반환값

- UnaryOperator<T> : Function의 자손. 매개변수 타입 = 반환 타입
- BinaryOperator<T> : BiFunction의 자손. 매개변수 타입 = 반환 타입

- 위의 내용은 generic으로 기본형은 사용할 수 없음. 기본현 특화 함수형 인터페이스를 사용하자. ex) IntPredicate
