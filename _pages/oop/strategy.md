---
layout: post
title:  "Strategy pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: strategy.md
---



Strategy Pattern
=======================


![pattern]({{site.baseurl}}/post_img/{{page.name}}/pattern.png)

- 전략을 쉽게 바꿀 수 있도록 해주는 디자인 패턴
   - 전략 : 어떤 목적을 달성하기 위해 일을 수행하는 방식, 비즈니스 규칙, 알고리즘 등

- 같은 문제를 해결하는 여러 알고리즘이 클래스별로 캡슐화되어 있고 이들을 필요에 따라 교체해서 사용할 수 있게 해준다

- 구성 요소
  - Strategy : 인터페이스나 추상클래스. 외부에서 동일한 방식으로 알고리즘을 호출하는 방식을 명시
  - ConcreteStrategy : Strategy를 실제로 구현한 클래스
  - Context : Strategy pattern을 실제로 이용하는 역할. 필요에 따라 전략을 변경할 수 있게 **setter** 가 존재

- - -

예시
===

```java
public abstract class Robot{
  private String name;
  public Robot(String name){
    this.name = name;
  }

  public String getName(){
    return this.name;
  }

  public abstract void attack();
  public abstract void move();
}


public class R1 extends Robot{
  public R1(String name){
    super(name);
  }

  public void attack(){
    System.out.println("R1 : Attactk");
  }

  public void move(){
    System.out.println("R1 : Move");
  }
}

public class R2 extends Robot{
  public R2(String name){
    super(name);
  }

  public void attack(){
    System.out.println("R2 : Attactk");
  }

  public void move(){
    System.out.println("R2 : Move");
  }
}
```

- 문제점 1 : R1에 새로운 기능을 추가하려면 abstract class를 수정해야 한다
- 문제점 2 : 새로운 로봇을 만들어서 기존의 로봇의 기능을 갖게 하려면 중복이 생긴다

- 해결책 : **변화되는 것** 을 찾아 클래스로 캡슐화 한다. 이러한 기능을 인터페이스를 통해 접근한다


```java
public abstract class Robot{
  private String name;
  private AttackStrategy attactStrategy;
  private MoveStrategy moveStrategy;

  public Robot(String name){
    this.name = name;
  }

  public String getName(){
    return this.name;
  }

  public abstract void attack(){
    attackStrategy.attack();    
  }
  public abstract void move(){
    moveStrategy.move();
  }
  public abstract void setAttackStrategy(AttackStrategy attackstrategy){
    this.attackStrategy = attackStrategy
  }
  public abstract void setMoveStrategy(MoveStrategy moveStrategy){
    this.moveStrategy = moveStrategy
  }
}


public class R1 extends Robot{
  public R1(String name){
    super(name);
  }
}

public class R2 extends Robot{
  public R2(String name){
    super(name);
  }
}
```

- 공력과 이동에 관한 전략은 다음과 같이 한다

```java
interface MoveStrategy{
  public void move();
}
interface AttackStrategy{
  public void attack();
}

public class A1 implements AttackStrategy{
  public void attack(){
    System.out.println("A1");
  }
}

public class A2 implements AttackStrategy{
  public void attack(){
    System.out.println("A2");
  }
}

public class M1 implements MoveStrategy{
  public void move(){
    System.out.println("M1");
  }
}

public class M2 implements MoveStrategy{
  public void move(){
    System.out.println("M2");
  }
}
```

- - -
