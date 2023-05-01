---
layout: post
title:  "Factory method Pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: template.md
---

팩토리 메서드 패턴
================

- 객체의 생성 코드를 별도의 클래스 혹은 메서드로 분리함으로써 객체 생성의 변화에 대비하는 패턴
- 상황에 따라 적적할 객체를 생성하는 코드는 자주 중복될 수 있다. 이 때, 객체 생성 코드를 따로 분리하게 된다면 생성의 방식의 변화를 이 코드의 변경만으로도 반영할 수 있다.

```java
public enem SchedulingStrategy{
  RESPONSE_TIME, THROUGHPUT, DYNAMIC
}

public class ScheduleFactory{
  public static ElevatorScheduler getScheduler(SchedulingStrategy strategy){
    switch(strategy){
      //생략
    }
  }
}
```

이를 사용하는 클래스는 다음과 같다

```java
public class ElevatorManager{
  private ShedulingStrategy strategy;
  //중략
  void RequestElevator(int dest, Direction direction){
    ElevatorScheduler scheduler = SchedulerFactory.getScheduler(strategy);
    //중략
  }
}
```

상속
----------

- 위의 팩토리 클래스를 이용하는 방법 대신 템플릿 메서드 패턴을 이용하여 팩토리 메서드를 구현할 수 있다.
- 하위 클래스에서 적합한 클래스 객체를 생성하는 방식

```java
public abstract class ElevatorManager{
  private ShedulingStrategy strategy;
  //중략
  void RequestElevator(int dest, Direction direction){
    ElevatorScheduler scheduler = getScheduler();
    //중략
  }

  protected abstract ElevatorScheduler getSheduler();
}

public class Manager1 extends ElevatorManager{
  @Override
  protected abstract ElevatorScheduler getSheduler(){
    return new Scheduler1();
  }
}

public class Manager2 extends ElevatorManager{
  @Override
  protected abstract ElevatorScheduler getSheduler(){
    return new Scheduler2();
  }
}
```


![structure]({{site.baseurl}}/post_img/{{page.name}}/structure.git)

- Product : 팩토리 메서드로 생성될 객체의 공통 인터페이스. 위의 경우에는 ElevatorScheduler이다.
- ConcreteProduct : Product를 구현한 클래스. Scheduler1, Scheduler2가 해당
- Creator : 팩토리 메서드를 가지는 클래스. ElevatorManager해당
- ConcreteCreator : Creator를 상속받은 클래스. Manager1, Manager2

- - -

참고자료
=========

- <https://gmlwjd9405.github.io/2018/08/07/factory-method-pattern.html>
