---
layout: post
title:  "generics, enums, annotation"
date:   2019-12-18 01:02:59
author: Inhyuk
category: java
name: java_generics.md
---

Generics
=========

- 메서드나 collection 클래스 등에 **컴파일 시의 타입 체크** (compile-time type check) 를 해주는 기능
- 객체의 타입 안정성을 높이고 형변환의 번거로움을 제거함

선언
------

- 클래스 옆에 **<T>** (type variable) 를 붙이면 된다

```java
class Gen<T>{
  T x;
  void setX(T x){
    this.x=x;
  }
  T getX(){return this.x;}
}

Gen<String> b = new Gen<T>();
Gen b = new Gen();//가능은 하지만 이렇게 하지 않도록 하자. Object 타입이 지정된다
```

- Gen<T> : 지네릭 클래스
- Gen : 원시타입(raw type)
- T : type variable. 참조형 타입만 올 수 있다(원시 타입은 **Wrapper class** 로 사용하자)

- static 멤버에는 T를 사용할 수 없음
- generic 배열의 참조변수를 선언하는 것은 가능. BUT 배열을 생성하는 것은 불가능

```java
T[] itemArr;//참조 변수는 가능

T[] toArray(){
  itemArr = new T[10];//불가능
}
```


객체 생성과 사용
----------------


- 두 T가 상속 관계에 있어도 다형성을 적용할 수 없다
- BUT T는 같고 원시클래스가 상속 관계에 있으면 가능
- 추론이 가능한 T는 생략 가능

```java
Box<Apple> x = new Box<Apple>;
Box<Apple> x = new Box<BlueApple>;//Apple과 BlueApple이 상속 관계여도 불가능

Box<Apple> y = new FruitBox<Apple>;//가능

Box<Apple> z = new FruitBox<>(); //생략 가능
```

제한된 generic class
------------------

- generic type에 **extends** 를 사용하면 특정 타입의 자손타입들만 사용 가능
- interface를 조상으로 가지는 generic type도 **implements** 대신 **extends** 사용

```java
class Box<T extends Fruit & Eatable>{
  //Fruit클래스의 자손이고 Eatable 인터페이스를 구현하는 T
}
```


와일드 카드
----------

- generic type이 다른 것만으로는 오버로딩 불가능(컴파일러가 컴파일 할때만 사용하고 제거하기 때문)
- **?** : 와일드 카드. 어떠한 타입도 가능

```java
<? extends T> // T와 그 자손들만 가능
<? super T > // T와 그 조상들만 가능
<?> //모든 타입이 가능
```

메서드
------

- generic method : 메서드의 선언부에 generic type이 선언된 메서드. 반환 타입 바로 앞에 generic type선언
- 매개변수가 같은 T를 사용해도 같은 type을 넣어줄 필요는 없음
- 타입 매개변수는 메서드 내에서만 사용하고 버릴 것이기 때문에(지역변수처럼) static 메서드도 generic 가능(static 멤버 변수는 불가능)

```java
static <T> void sort(){

}
```

- - -

enums(열거형)
================

- 관련된 상수를 편리하게 선언하기 위한 것. 여러 상수를 정의할 때 사용
- 열거형은 **==** 사용 가능(타입까지 체크)
- **>, <** 는 사용 불가능

```java
class Card{
  enum Kind{CLOVER,HEART};
  enum Value{TWO,THREE,FOUR};
  void move(){
    switch(dir){
      case clover ://열거형의 이름은 빼고 상수 이름만 적어야 함
        x++;
        break;
      case heart:
        x--;
        break;
    }
  }
}
```

- Enum 클래스이 메서드
  - String name() : 상수의 이름을 문자열로 반환
  - int ordinal() : 상수가 정의된 순서를 반환
  - T valueOf(String name) : name과 일치하는 열거형 상수 반화
  - values() : 모든 상수를 배열에 담아 반환

```java
for(Kind k : Kind.values()){
  //code
}
System.out.println(Kind.CLOVER == Kind.valueOf("CLOVER"));//true
```

멤버 추가
--------

- 열거형 상수를 다른 값과 연관시켜 사용할 수 있음(기본은 0부터 순서대로 부여)

```java
enum Direction{
  EAST(1),SOUTH(5),WEST(-1),NORTH(3);

  final private int val;
  Direction(int val){
    this.val= val;
  }  
  public int getVal(){
    return this.val;
  }
}

System.out.println(Direction.WEST.getVal());//result :-1;
```

- - -

annotation
=============

- 프로그램에 영향은 미치지 않지만 다른 프로그램에게 유용한 정보를 제공
- 속성을 부여할 수 있음

```java
@annotation
//밑에는 변수, 클래스, 메서드 등이 위치

@annotation(속성1=값,속성2=값)
@annotation(값)//하나의 속성만 가질때는 줄여쓰기 가능
```

표준 annotation
---------------

- 자바에서 기본적으로 제공하는 annotation
- meta annotation : annotation을 정의하는데에 사용하는 annotation

#### 1. @Override

- 부모 클래스의 메서드가 오버라이드 되었다는 것을 컴파일러에게 알려줌
- 오타등으로 인해 매칭되는 메서드가 없으면 에러 발생

#### 2. @Deprecated

- 해당 클래스 혹은 메서드가 더이상 지원하지 않는다는 것을 알려줌
- 컴파일시 경고 메시지를 날려준다

#### 3. @FunctionalInterface

- 함수형 인터페이스라는 것을 알려줌
- 컴파일러가 확인하여 함수형 인터페이스로 선언되었는지를 확인

#### 4. @SuppressWarnings

- 컴파일러의 경고 메시지를 보여주지 않도록 함
- 특정 메시지를 골라서 안보이게 할 수도 있음
  - deprecation : deprecated가 붙은 대상의 메시지 안보이게 함
  - unchecked : 지네릭스의 타입을 정하지 않았다는 경고 메시지를 안보이게 함
  - rawtypes : 지네릭스를 사용하지 않아서 발생하는 경고를 안보이게 함
  - varargs : 가변인자 타입이 지네릭스 타입일 때 발생하는 경고를 안보이게 함


```java
@SuppressWarnings("unchecked")//지네릭스 관련 경고 억제
ArrayList list = new ArrayList();
```

meta annotation
-----------------

- annotation을 위한 annotation
- annotation의 유지기간, 적용대상 등을 정의

#### 1. @Target

- annotation이 적용가능한 대상을 지정
- 적용 대상 종류
  - ElementType.ANNOTATION_TYPE : annotation
  - ElementType.CONSTRUCTOR : 생성자
  - ElementType.FIELD : 필드(멤버변수, enum상수)
  - ElementType.LOCAL_VARIABLE : 지역변수
  - ElementType.METHOD : 메서드
  - ElementType.PACKAGE : 패키지
  - ElementType.PARAMETER : 매개변수
  - ElementType.TYPE : 타입(클래스, 인터페이스, enum)
  - ElementType.TYPE_PARAMETER : 타입 매개변수
  - ElementType.TYPE_USE : 타입이 사용되는 모든 곳

#### 2. @Retention

- 유지되는 기간을 지정
- RetentionPolicy.SOURCE : 컴파일전까지 유효. 소스파일에만 존재
- RetentionPolicy.CLASS : 클래스 파일에 존재. 실행시 사용 불가. 기본 값
- RetentionPolicy.RUNTIME : 클래스 파일에 존재. 실행시 사용 가능

```java
//인터페이스 만들기
@Target({ElementType.METHOD,ElementType.FIELD})//대상 정의
@Retention(RetentionPolicy.SOURCE)//적용 기간
@interface MyAnnotation{

}
```

#### 3. @Documented

- javadoc으로 작성한 문서에 포함되도록 함
- @Override, @SuppressWarnings 를 제외하고 모두 붙어 있음

#### 4. @Inherited

- annotation이 자손 클래스에 상속되도록 함

#### 5. @Repeatable

- java8부터 지원
- 여러 개의 annotation적용 가능하게 함(보통은 하나의 대상에 하나의 annotation)

직접 만들기
------

```java
@interface annotation_name{
  type element();
}
```

- element : annotation 내에 선언된 method. 반환값은 있고 매개변수는 없음. annotation 적용시 모든 값을 지정해주어야 함
- element에 기본값을 적용가능. 기본값 지정시 생략 가능

```java
@interface MyAnnotation{
  int count() default 3;
  String name();
  String[] tools();
}

@MyAnnotation(
  count=4,
  name="JiSun",
  tools={"java","python"}
)
class NewClass{

}
```
