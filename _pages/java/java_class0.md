---
layout: post
title:  "클래스, 상속"
date:   2019-12-18 01:02:59
author: Inhyuk
category: java
name: java_class0.md
---

클래스의 변수
============

- 클래스 변수, 인스턴스 변수, 지역변수로 나뉨

1. 인스턴스 변수

- 객체가 생성시에 독립적인 공간에 만들어지는 변수
- 객체마다 변수가 다 다르게 부여

2. 클래스 변수

- 클래스가 로딩시 생성되는 변수. 프로그램이 종료될 때 까지 유지
- 같은 클래스는 같은 변수를 공유한다

3. 지역 변수

- 매서드 내부, 초기화 블럭 등에서 일시적으로 사용되는 변수
- 작업이 끝나면 사라진다


- - -

매개변수
======

1. 기본형 매개변수

- 변수의 값을 읽기만 한다

2. 참조형 매개변수

- 변수의 값을 읽고 변경 가능.
- 메모리의 주소 값을 가져온다
- 참조형 return값은 그 값의 주소를 반환한다

- - -

클래스 메서드 & 인스턴스 메서드
======

1. 클래스 메서드

- 메서드 앞에 **static** 이 붙어 있는 메서드
- 객체를 생성하지 않고도 사용 가능
- 모든 객체에서 공통으로 사용하는 메서드는 클래스 메서드로 선언한다
- 인스턴스 변수를 사용할 수 없다
- 인스턴스 변수를 사용하지 않으면 클래스 메서드로 선언하는 것이 좋음

2. 인스턴스 메서드

- 인스턴스 변수와 관련된 일을 하는 메서드
- 객체를 생성을 해야 사용할 수 있다


- - -

생성자
=====

- 생성자는 이름이 클래스 이름과 같아야 함
- 반환값이 없음
- 여러가지 생성자를 오버로딩 가능
- 다른 메서드 혹은 다른 생성자에서 생성자를 호출시 **this()** or **this(args)** 를 사용한다.
- 이 객체의 멤버변수는 **this.변수** 로 접근 가능

```java
//생성자를 이용한 다른 변수 복사
class Car{
  int price;
  Car(){
    this.price=0;
  }
  Car(Car c){
    this.price=c.price;
  }
}
```

- - -

초기화
=====

- 변수의 초기화의 순서
  1. 기본값
  2. 명시적 초기화
  3. 초기화 블럭 : 복잡한 초기화에 주로 사용. 자식 클래스에 상속되지 않기를 바라는 초기화에도 사용
  4. (인스턴스 변수이면) 생성자

- 클래스 변수는 클래스가 로딩 시 한번 초기화 된다
- 인스턴스 변수는 객체가 생성될 때마다 각 객체에서 초기화된다.

```java
class Car{
  static int value=4;//명시적 초기화
  int price=4;//명시적 초기화

  static{//static 초기화 블럭
    value=3;  
  }
  {//인스턴스 변수 초기화
    price=3;
  }

  Car{
    this.price=5;
  }
}
```

- - -

상속
====

- 다른 클래스(부모 클래스)의 변수, 메서드를 새로운 클래스(자식 클래스)에서 사용할 수 있게 하는 기능
- 생성자와 초기화 블럭은 상속되지 않음
- [**자식클래스** is a **부모클래스**] 의 관계.
- **Object 클래스** : 모든 클래스의 조상 클래스. 명시적으로 상속을 지정하지 않으면, Object 클래스를 상속한다

```
class Parent{
  int x=3;
}

class Child extends Parent{

}
```

오버라이딩
------------

- 상속받은 클래스의 기존 메서드 대신하여 같은 이름의 다른 메서드를 만드는 것
- 메서드 이름, 리턴 타입, 매개변수가 같아야 한다.
- 부모 메서드의 접근 제어자보다 좁은 제어자를 지정할 수 없다.
- 부모 메서드의 예외보다 많은 수의 예외를 선언할 수 없음

super
------

- 부모 클래스의 멤버들을 참조하는 방법
- 상속을 받게 되면 부모의 멤버를 사용할 수 있지만, 같은 이름의 변수를 다르게 사용하거나 오버라이딩 되기 전 메서드를 호출하고 싶으면 사용한다
- **super()** 를 통해서 부모 클래스의 생성자 호출 가능. 없을시 부모의 default 생성자를 자동으로 생성자 첫 줄에 삽입한다

- - -

제어자(modifier)
==============

- 접근제어자 : 변수, 클래스, 객체, 메서드 등에 접근할 수 있는 범위 지정. public, protected, default, private
- 그외 : static, abstract, final 등

- 메서드에 static + abstract 불가능
- 클래스에 abstract + final 불가능
- abstract은 private 불가능
- 메서드에서는 private와 final을 같이 쓸 필요 없음.

static
------

- 공통으로 사용한다는 뜻
- 객체를 생성하지 않고 사용할 수 있음

final
------

- final 변수 : 값을 변경할 수 없는 상수. 초기화는 가능. 인스턴스 변수인 경우에는 생성자에서는 초기화 가능
- final 클래스 : 변경될 수 없는 클래스. 확장될 수 없기 때문에 다른 클래스의 부모 클래스가 될 수 없음
- final 메서드 : 오버라이딩 불가

abstract
--------

- 추상 클래스에 사용
- 선언은 하지만, 구현은 하지 않는다는 의미

접근 제어자
---------

1. private : 같은 클래스 내에서만 접근 가능. private 메서드는 오버라이딩 불가
2. default : 같은 패키지 내에서만 접근 가능. 특정 접근제어자를 명시하지 않을 시에도 적용.
3. protected : 같은 패키지 내 혹은 다른 패키지에서의 상속한 클래스에서 접근 가능
4. public : 제한 없음

- - -

다형성
=====

- 여러가지 형태를 가질 수 있는 능력
- 자바에서는 한 타입의 참조변수로 다른 타입의 객체들을 참조&사용할 수 있도록 하는 것을 의미
- 조상 클래스에서 명시적 형변환 없이 자식 클래스의 객체를 사용할 수 있도록 하는 것
- 오버라이딩이 된 경우에는 오직 **인스턴스 메서드** 만이  참조 변수의 타입에 관계 없이 호출된다. 나머지는 참조 변수의 타입에 따라 달라진다

```java
class a{
    public static void main(String[] args){
        Parent p = new Child();//가능
        Child c = new Parent();//불가능
        p.childVar;//불가능

        System.out.println(p.x);//result : 3
        p.printing();//result : This is a parent class

        //명시적 형변환은 가능
        Child c = (Child)p;

        System.out.println(c.x);//result : 4
        c.printing();//result : This is a child class
    }
}

class Parent{
    int x=3;
    void printing(){
        System.out.println("This is a parent class");
    }
}

class Child extends Parent{
    int x=4;
    void printing(){
        System.out.println("This is a child class");
    }
}
```

- (명시적이든 아니든) 형변환을 하게 되면 인스턴스 자체의 형을 변환하는 것이 아니기 때문에 인스턴스 자체에는 영향 X.

```java
class Parent{}
class Child1 extends Parent{}
class Child2 extends Parent{}

class check{
  void doSomething(Car c){
    if(c instanceof Child1){
      //
    }  
    else if(c instanceof Child2){
        //
    }  
  }
}
```

- **instanceof** 는 형변환이 가능한지를 알려준다.

- 다음과 같이 매개변수에도 사용할 수 있다.

```java
class Product{
  int price;
}
class TV extends Product{}
class PC extends Product{}
class Phone extends Product{}

int discountPrice(Product p, int dis){
  return p.price- dis;//각 클래스마다 메서드를 만들 필요 없게 된다.
}
```
