---
layout: post
title:  "Template method Pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: template.md
---

템플릿 메서드 패턴
==================

- 전체적으로 동일하면서 부분적으로는 다른 구문으로 구성된 메서드의 코드 중복을 최소화 할때 사용
- 동일한 기능을 상위 클래스에서 정의하면서 확장, 변화가 필요한 부문만 서브 클래스에서 구현할 수 있게 함
- 템플릿 메서드 패턴은 전체적인 알고리즘은 상위 클래스에서 구현하고 다른 부분만 하위 클래스에서 구현하게 하는 디자인 패턴

- 구성
  - AbstractClass : 템플릿 메서드를 정의하는 클래스. 하위 클래스에 공통 알고리즘을 정의
  - ConcreteClass : 공통 알고리즘의 다른 부분을 구현하는 클래스


```java
public abstract class Motor{
  private Door door;
  private MotorStatus motorStatus;

  public Motor(Door door){
    this.door = door;
  }

  //중략
  private void move(Direction direction){
    if (motorStatus == MotorStatus.MOVING){
      return;
    }
    moveMotor(direction);
  }

  protected abstract void moveMotor(Direction direction);
}

public class M1 extends Motor{
  protected void moveMotor(Direction direction){
    //다른 부분
  }
}

public class M2 extends Motor{
  protected void moveMotor(Direction direction){
    //다른 부분
  }
}
```
