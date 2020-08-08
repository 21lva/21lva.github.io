---
layout: post
title:  "Command Pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: command.md
---

Command pattern
===============

![structure]({{site.baseurl}}/post_img/{{page.name}}/structure.gif)

- 이벤트가 발생했을 때 실행될 기능이 다양하고 변경이 필요한 경우 사용
- 요청 자체르 캨슐화 한다.
- 실행되는 기능을 캡슐화해서 호출자 클래스와 기능을 실행하는 클래스 사이의 의존성을 제거.
- 외부에서 봤을 때는 어떤 객체가 reciever의 역할을 하는지 알 수가 없다.

- 구성
  - Command : 실행될 기능에 대한 인터페이스. 기능을 **execute** 메서드로 선언
  - ConcreteCommand : 실제로 기능을 구현한 클래스
  - Invoker : 기능의 실행을 요청하는 클래스. 호출자 클래스
  - Receiver : ConcreteCommand에서 execute 메서드를 구현할 때 필요한 클래스. Concrete Command의 기능을 실행하기 위해 사용하는 수신자 클래스

- 리시버가 바로 command 인터페이스를 구현하지 않는 이유 : 리시버가 인터페이스를 직접 구현하고 행동에 대한 다른 기능을 추가하게 되면 기능 단위로 나뉘어진 클래스인 리시버에 해당 기능을 추가해야 한다. 또한, 직접 구현을 하지 않게 되면 리시버 클래스의 특정 기능만을 담당하는 인터페이스 같은 역할을 하는 ConcreteCommand 객체를 여럿 생성할 수 있다.

Command와 ConcreteCommand의 모습

```java
interface Command{
  void execute();
}

class LightOnCommand implements Command{
  Light light;

  public LightOnCommand(Light light){
    this.light = light;
  }

  public void execute(){
    light.on();
  }
}
```

invoker 클래스

```java
class RemoteControl{
  Command slot;

  public void setCommand(Command command){
    this.slot = command;
  }

  public void pressButton(){
    slot.execute();
  }
}
```

- - -

vs Strategy pattern
=================

[stackoverflow 답변](https://stackoverflow.com/questions/4834979/difference-between-strategy-pattern-and-command-pattern)

- Command pattern은 동작되어야 하는 **무엇** 을 캡슐화 한 것이다.
  - ex) 리모컨, 로그 등등
- Strategy pattern은 **어떻게** 동작되는 지를 캡슐화 한 것.
  - ex) 정렬 알고리즘 등
- 쉽게 말하면 strategy pattern은 **전략** 이 바뀌는 경우, command pattern은 **행동** 이 바뀌는 경우에 대한 설계이다.

- - -

예시
====

버튼이 눌렸을 때 램프 불이 켜지는 프로그램을 개발하려고 한다.

이 때 필요한 사안은 버튼을 인식하는 클래스와 실제 불을 켜는 클래스이다.

```java
public class Lamp{
  public void turnOn(){
    System.out.println("Turn On");
  }
}

public class Button{
  private Lamp lamp;

  public Button(Lamp lamp){
    this.lamp = lamp;
  }

  public void press(){
    lamp.turnOn();
  }
}
```

위의 구조는 문제점이 존재한다
  - 램프가 아닌 다른 기능을 하도록 변경하게 하려면?
  - 버튼을 누르는 행위에 다른 기능을 추가하려면?

첫 문제를 해결하려고 다음과 같은 코드를 작성했다

```java
public class Alarm{
  public void turnOn(){
    System.out.println("Turn On");
  }
}

public class Button{
  private Alarm alarm;

  public Button(Alarm alarm){
    this.alarm = alarm;
  }

  public void press(){
    alarm.turnOn();
  }
}
```

많은 부분이 변경되었다. OCP에 위배된다는 말이다.

버튼을 누르는 경우에 따른 기능을 추가하려 해도 OCP에 위배될 수 밖에 없다.

이제 이 문제들을 해결하기 위하여 Command Pattern을 적용할 수 있다.
