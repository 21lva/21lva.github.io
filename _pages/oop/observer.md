---
layout: post
title:  "Observer Pattern"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: observer.md
---


예시
===

성적을 출력하는 기능을 프로그램으로 작성해보자.

이럴 경우에은 2가지 클래스를 만들 수 있다.
  - ScoreRecord : 점수를 저장하는 클래스
  - DataSheetView : 점수를 목록으로 출력하는 클래스

성적이 입력되는 경우에는 ScoreRecord의 addScore이 호출되고 이를 위해 DataSheetView를 참조하고 있어야 한다.
  - addScore -> DataSheetView.update

DataSheetView의 경우에도 ScoreRecord를 참조해서 getScoreBoard을 호출하여 출력할 점수를 얻어낸다.

문제점
-----

하지만 위의 구조는 다음과 같은 문제점이 존재한다
  - 성적 출력형태를 변경하고 싶으면?
  - 여러 가지 형태의 성적을 동시 또는 순차적으로 출력하고 싶으면?

예를 들어, 순차적 표시가 아닌 최대값으로 표시하고자 한다면, DataSheetView대신 MinMaxView 클래스를 추가해야한다. 이는 OCP에 위배된다.

해결책
-----

- 이 문제를 해결하는 방법의 핵심은 통보 방식이 변경되더라도 ScoreBoard 클래스는 지속적으로 재사용 될 수 있어야 한다는 것이다.
- 위의 문제의 경우 ScoreBoard의 update가 발생하면 대상에게 통보하는 기능을 인터페이스로 일반화 하는 것이 좋다

- 구성 요소
  - Subject : attach와 detach 메서드를 통해서 관심 대상을 추가, 제거한다.
  - observer : subject 클래스에서 관리하는 대상들에 대한 인터페이스. DataSheetView, MinMaxView등이 이 인터페이스의 update 메서드를 구현한다
  - ScoreBoad : Subject를 상속한다
  - DataSheetView, MinMaxView

아래는 다음을 구현한 코드다

```java
public interface Observer{
  public void update();
}

public abstract class Subject{
  private List<Observer> observers = new ArrayList<>();

  public void attach(Observer observer){
    observers.add(observer);
  }

  public void detach(Observer observer){
    observers.remove(observer);
  }

  public void notify(){
    for(Observers observer : observers){
      observer.update();
    }
  }
}

public class ScoreRecord extends Subject{
  private List<Integer> scores = new ArrayList<>();
  public void addScore(int score){
    //생략
    notify();
  }

  public List<Integer> getScoreBoard(){
    return scores;
  }

}

public class DataSheetView implements Observer{
  private ScoreBoard scoreBoard;
  //생략  
}

public class MinMaxView implements Observer{
  private ScoreBoard scoreBoard;
  //생략  
}
```

- - -


옵저버 패턴
==========

![structure]({{site.baseurl}}/post_img/{{page.name}}/structure.jpg)

- 데이터의 변경이 발생했을 경우 상대에 의존하지 않으면서 데이터의 변경을 통보받는 패턴
- 대상을 관리하는 subject 추상 클래스와 정보를 전달 받는 Observer 인터페이스로 나눠서 일반화 한다. 이렇게 하면, subject 클래스를 상속받는 클래스의 관리 대상들에 대한 의존성을 없앨 수가 있다.


- 구성
  - observer : 데이터의 변경을 통보 받는 인터페이스
  - subject : ConcreteObserver 객체를 관리하는 요소. 실제 통보를 하는 객체가 상속 받는 클래스이다.
  - ConcreteSubject : 변경을 통보하는 클래스.
  - ConcreteObserver : 통보를 받는 클래스. update 메서드를 구현해서 통보 받은 데이터를 가지고 작업을 수행한다. ConcreteSubject의 getState를 통해서 원하는 정보를 얻어낸다.
