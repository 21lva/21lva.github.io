---
layout: post
title:  "Abstract Factory Pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: abstractFactory.md
---


추상 팩토리 패턴
============

추상 팩토리 패턴은 관련성이 있는 여러 종류의 객체를 일관된 방식으로 생성하는 경우에 유용하다.


![structure]({{site.baseurl}}/post_img/{{page.name}}/structure.png)

- 구조
  - AbstractFactory : 실제 팩토리 클래스의 공통 인터페이스.
  - ConcreteFactory : AbstractFactory를 구현하는 구체적인 팩토리 클래스다. 연관성 있는 객체들을 묶어서 생성하는 역할을 한다
  - AbstractProduct : 제품의 공통 인터페이스. 각 연관성 있는 객체들의 묶음에서 한 역할을 담당하는 객체들에 대한 인터페이스
  - ConcreteProduct : AbstractProduct를 구현하는 클래스

- - -

예제
========

엘리베이터를 만드는 경우를 생각해보자.

Elevator 클래스에는 Motor와 Door 객체를 포함한다.

Motor 클래스와 Door클래스의 모습

```java
public abstract class Motor{
  private MotorStatus status;

  //생략
}

public class MotorA extends Motor{
  //생략  
}

public class MotorB extends Motor{
  //생략  
}

public abstract class Door{
  //생략
}

public class DoorA extends Door{
  //생략  
}

public class DoorB extends Door{
  //생략  
}
```

위의 구조에 팩토리 메서드 패턴을 적용해보자

```java
public class MotorFactory{
  public static Motor create(Vendor vendor){
    switch (vendor){
      case A:
        return new MotorA();
      case B:
        return new MotorB();
    }
  }
}

public class DoorFactory{
  public static Door create(Vendor vendor){
    switch (vendor){
      case A:
        return new DoorA();
      case B:
        return new DoorB();
    }
  }
}
```

이를 이용하는 클라이언트는 다음과 같다

```java
public class Client{
  public static void main(String[] args){
    Door aDoor = DoorFactory.create(Vendor.A);
    Door aMotor = MotorFactory.create(Vendor.A);

    //생략
  }
}
```

여기서의 문제점은 다음과 같다.
- 새로운 부품이 추가된다면, 클라이언트의 구현 구조가 복잡해지고, 새로운 팩토리 클래스를 추가해야한다
- 새로운 업체가 생성된다면 많은 부분을 수정해야 한다.

이를 해결하기 위해 추상 팩토리 패턴을 적용해보자.

보통 각 업체는 각자의 엘리베이터를 만들기 위해 각자의 부품을 사용하므로 부품 객체들 사이에는 연관성이 존재한다.

연관성 있는 객체들을 묶어서 생성하는 팩토리 클래스를 만들자

```java
public abstract class ElevatorFactory{
  public abstract Motor createMotor();
  public abstract Door createDoor();
}

public class ElevatorFactoryA extends ElevatorFactory{
  public Motor createMotor(){
    return new MotorA();
  }

  public Door createDoor(){
    return new DoorA();
  }
}

public class ElevatorFactoryB extends ElevatorFactory{
  public Motor createMotor(){
    return new MotorB();
  }

  public Door createDoor(){
    return new DoorB();
  }
}
```

이를 사용하는 클라이언트 클래스는 다음과 같다

```java
public class Client{
  public static void main(String[] args){
    String vendor = args[0];
    ElevatorFactory factory = null;
    if(vendor = Vendor.A) {
      factory = new ElevatorFactoryA();  
    } else {
      factory = new ElevatorFactoryB();  
    }
    //생략
  }
}
```

새로운 업체가 등장하는 경우에는 새로운  ConcreteFactory를 구현하면 된다


또한, Factory 객체는 1개만 존재하면 된다. 이를 위해 싱글턴 패턴을 적용해보자

```java
public class ElevatorFactoryA extends ElevatorFactory{
  private static ElevatorFactory factory;

  public static ElevatorFactory getInstance(){
    if (factory == null) {
      factory = new ElevatoryFactoryA();
    }

    return factory;
  }
  public Motor createMotor(){
    return new MotorA();
  }

  public Door createDoor(){
    return new DoorA();
  }
}
```
