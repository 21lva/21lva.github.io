---
layout: post
title:  "디자인 패턴"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: design_pattern.md
---

디자인 패턴
=========

- 소프트웨어를 설계할 때 특정 맥락에서 자주 발생하는 문제들을 해결할 수 있는 해결책.
- 소프트웨어의 경우에는 다양한 소프트웨어를 개발할 때 공통되는 설계 문제가 존재하고 그 문제를 해결하는 데에 비슷한 해결방법을 적용할 수 있다.
- 디자인 패턴은 다음과 같이 말할 수 있다

```
바퀴를 다시 발명하지 마라
```

- 패턴 : 비슷한 상황이나 유형들이 반복되어 나타나는 것을 의미.
  - context : 문제가 발생하는 여러 상황. 패턴이 적용될 수 있는 상황
  - 문제 : 패턴이 적용되어 해결될 필요가 있는 이슈
  - 해결 : 문제를 해결하도록 하는 구성 요소들과 그 요소들 사이의 관계, 책임 등을 기술

- - -

GoF 디자인 패턴
================

- 디자인 패턴을 23가지로 정리.
  - 생성 : 객체 생성과 관련되 패턴. 객체의 생성과 조합을 캡슐화해 특정 객체가 생성되거나 변경되어도 구조에 문제가 없도록 유연성을 제공. **factory method, abstract factory, builder, prototype, singleton**
  - 구조 : 클래스나 객체를 조합해 더 큰 구조를 만드는 패턴. **adapter, bridge, composite, decorator, facade, flyweight, proxy**
  - 행위 : 객체나 클래스 사이의 알고리즘이나 책임 분배와 관련된 패턴. 객체의 결합도를 최소화 하면서 객체에 책임을 분배하는 방식에 관련됨. **command, interpreter, iterator, mediator, memento, observer, state, strategy, templete method, visitor**

- - -

UML
======

- UML에는 디자인 패턴을 표현하는 **Collaboration** 이 있음. 구조적인 면과 행위적인 면을 모두 표현

Collaboration
------------

- 디자인 패턴을 역할들의 상호협동 작업으로 간주할 수 있다.
- UML에서는 객체들이 특정 상황에서 수행하는 역할의 상호작요을 collaboration으로 작성
- 역할들의 상호작용을 추상화한 것으로, 특별한 상황에 재사용 가능.
- Collaboration occurrence : 구체적인 상황에서의 collaboration을 보여준다

순차 다이어그램
---------------------

- 객체들의 상호작용을 나타내는 다이어그램
- 객체들 사이의 메시지 송신과 그들의 순서를 나타낸다.

![순차 다이어그램]({{site.baseurl}}/post_img/{{page.name}}/diagram.JPG)

- 점선 : 생명선. 해당 객체의 존재를 나타냄
- 사각형 박스 : 활성 구간. 연산을 실행하는 시간
- 실선 화살표 : 객체 사이의 메시지 전달. 빈 화살표는 비동기 메시지 전달이다
  - 화살표 위에 [시퀀스 번호][가드]: 반환값:=메시지이름([인자]) 로 표시된다
  - 가드 : 메시지가 송신되기 위해 만족해야 하는 것
- 점선 화살표 : 응답 메시지
