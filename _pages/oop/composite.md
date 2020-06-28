---
layout: post
title:  "Compoiste Pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: composite.md
---

Compoisite Pattern
================

**부분-전체의 합성(Composite) 관계** 를 가지는 객체들을 정의할 때 유용

특정 부분을 하나 하나 추가하는 것이 아닌 부분들의 부모 클래스나 인터페이스를 포함하도록 해서 OCP를 유지한다


![structure]({{site.baseurl}}/post_img/{{page.name}}/structure.png)

- 구조
  - Component : 구체적으로 클래스를 정의하는 클래스의 인터페이스 혹은 추상클래스
  - Leaf : 구체적인 클래스. Component를 구현하거나 상속하고, Composite 클래스의 구성 요소가 된다
  - Composite : Leaf를 포함하는 전체 클래스

- - -

예제
====

컴퓨터를 구성요소로 포함한다고 하자. 이 때 다음과 같이 설계를 하면 문제가 발생한다

```java
public class Keyboard{
  //생략
}

public class Monitor{
  //생략
}

public class Mouse{
  //생략
}

public class Computer{
  private Monitor monitor;
  private Mouse mouse;
  private Keyboard keyboard;

  //생략  
}
```

위 구조의 문제점은 새로운 부품인 Speaker가 추가되는 경우에 Computer 클래스를 수정해야 한다.

이 문제를 해결하기 위해 다음과 같이 **Compoiste Pattern** 을 적용할 수 있다

```java
public abstract class Device{
  int price;
  public int getPrice() {
    return price;
  }

}


public class Keyboard extends Device{
  //생략
}

public class Monitor extends Device{
  //생략
}

public class Mouse extends Device{
  //생략
}

public class Computer{
  private List<Device> devices;
  //중략


  public addDevice(Device newDevice) {
    devices.add(newDevice);
  }

  public getTotalPrice() {
    return devices.stream().map(Device::getPrice).reduce(Math::sum).orElse(0);
  }
}
```

여기에 만약에 Speaker 클래스가 추가 된다면 Computer 클래스를 수정할 필요가 없어진다.
