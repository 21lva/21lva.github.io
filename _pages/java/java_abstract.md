---
layout: post
title:  "추상클래스 & 인터페이스"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_abstract.md
---

추상 클래스
==========

- 자체로서는 클래스의 역할을 다 하지 못하지만 다른 클래스의 바탕이 되는 클래스
- 클래스 앞에 **abstract** 키워드를 붙인다
- 특정 메서드에 **abstract** 를 붙여서 미완성 상태로 두고 자식 클래스가 구현하도록 한다
- 객체를 생성할 수 없다. 모든 메서드는 구현되어 있지만 클래스에만 abstract를 붙여서 객체 생성을 막을 수 있다

추상메서드
---------

- 메서드 앞에 **abstract** 를 붙이고 구현은 하지 않은 메서드
- 추상클래스를 상속 받은 자식 클래스는 모두 구현하거나 자신도 추상 클래스가 된다

```java
abstract class Parent{
  int value;
  void notStatic(){
    //구현
    //추상 클래스에도 추상 메서드가 아닌 메서드를 만들 수 있다
  }
  abstract void play(int pos);
  abstract void stop();
}

class Child1 extends Parent{
  void play(int pos){
    //구현부
  }
  void stop(){
    //구현부
  }
}

abstract class Child2 extends Parent{
  void play(int pos){
    //구현부
  }  
}
```

- - -

인터페이스
=========

- 추상 클래스처럼 구현을 하지 않은 상태의 메서드를 가진 것
- 추상 클래스 : 미완성 설계도 vs 인터페이스 : 밑그림
- 추상 메서드(**public abstrac** )와 상수(**public final** )만을 가진다.
- 객체 생성 불가능

```java
interface Readable{
  public final int=3;
  public abstract void play(int pos);
}
```

- 인터페이스는 추상 클래스와 달리 다중 상속이 가능하다
- 자식 클래스에서 인터페이스의 모든 메서드를 구현하지 않은 경우에는 추상 클래스가 된다

```java
interface Readable{
  public final int=3;
  public abstract void play(int pos);
  public abstract void play2(int pos);
}

class Read implements Readable{
  public void play(int pos){
    //구현
  }
  public void play2(int pos){
    //구현
  }
}

abstract class Read implements Readable{
  public void play(int pos){
    //구현
  }
}
```

장점
----

- 개발시간을 단축
- 표준화가 가능
- 서로 관계 없는 클래스들을 묶을 수 있음
- 독립적인 프로그래밍 가능. 구현부와 선언부가 다르기 때문에 구현부가 바뀌어도 선언부를 가져다 쓰는 다른 코드에서는 신경안써도 됨

default method
---------------

- Java8 에서 추가된 기능
- 기존에는 인터페이스에 새로운 메서드가 추가되면 이를 구현한 모든 클래스에서 메서드를 구현해야 함
- 메서드 앞에 **default** 키워드를 붙여서 메서드를 만들고 구현을 한다
- 접근제어자는 **public** 이고 생략가능
- 원하면 오버라이딩 생략가능

```java
public class iii {
    public static void main(String[] args) {
        Lion l= new Lion();
        Monkey m= new Monkey();
        feed(l);
        feed(m);
        l.fun();
        m.fun();
    }
    public static void feed(animal a){
        a.getFood();
    }
}

interface animal{
    public abstract void getFood();
    default void fun(){
        System.out.println("default method");
    }
}

class Lion implements animal{
    public void getFood(){
        System.out.println("Get chickens");
    }
}

class Monkey implements animal{
    public void getFood(){
        System.out.println("Get bananas");
    }
    public void fun(){
        System.out.println("Monkey overrided it");
    }
}
```


- - -

추상클래스 vs 인터페이스
=====================

인터페이스
---------

- 특정 기능이 존재함을 보장해준다. 하나의 규약이다.
- 다중 상속이 가능

추상 클래스
-----------

- 인터페이스랑 달리 일반 메서드가 존재한다
- 변수를 가지면서 클래스 상태가 존재한다
- 추상클래스는 비슷한 행동을 하는 클래스끼리 묶는 것이다. (**is-a**)
