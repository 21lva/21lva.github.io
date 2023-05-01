---
layout: post
title:  "Decorator Pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: decorator.md
---

Decorator pattern
=================

- 기본 기능에 추가할 수 있는 기능의 종류가 많은 경우에 각 추가 기능을 **Decorator** 클래스로 정의한 후 필요한 것들만 조합해서 사용하는 설계 방식

![structure]({{site.baseurl}}/post_img/{{page.name}}/structure.png)

- 구성
  - Component : ConcreteComponent와 DecoratorComponent들이 구현해야 하는 기능을 정의.
  - ConcreteComponent : 기본 기능을 구현한 클래스
  - Decorator : Decorator의 구체적인 기능을 묶음
  - ConcreteDecorator : Decorator의 하위 클래스. 기본 기능에 추가되는 개별적인 기능

```java
public interface Display {
  void draw();
}

public class BasicDisplay implements Display{
  @Override
  public void draw() {
    System.out.println("Basic Display");
  }
}
```

데코레이터의 모습

```java
public abstract class DisplayDecorator implements Display{
  private Display display;

  DisplayDecorator(Display display){
    this.display = display;
  }

  public void draw(){
    display.draw();
  }
}

public class D1 extends DisplayDecorator{
  D1(Display display) {
    super(display);
  }

  @Override
  public void draw(){
    super.draw();
    System.out.println("D1");
  }
}

public class D2 extends DisplayDecorator{
  D2(Display display) {
    super(display);
  }

  @Override
  public void draw(){
    super.draw();
    System.out.println("D2");
  }
}
```

실행을 위한 메인 클래스

```java
public class main {
  public static void main(String[] args){
    Display display = new D2(new D1(new BasicDisplay()));

    display.draw();
  }
}
```

결과

```
Basic Display
D1
D2
```
