---
layout: post
title:  "WebFlux + Netty"
date:   2019-08-10 02:02:59
author: Inhyuk
category:
tags:
cover:  "/assets/instacode.png"
name: webflux.md
---

Netty
=====

- Tomcat의 경우에 쓰레드 풀의 쓰레드 갯수가 200
- Jetty는 8에서 200개
- netty는 core*2의 Thread만 생성

- Spring MVC 동작 방식(Tomcat)
1. 요청이 들어오면 ThreaPool의 thread사용
2. I/O interruption 발생 시 CPU를 block(idle 상태)
3. 다른 요청이 들어오면 1시행
4. 2의 block이 풀리면 작업 이어나감

- netty에서는 발생한 이벤트를 event queue에 넣고 event loop가 꺼내서 동작하게 만듬
- 단일 스레드 이벤트 루프
  - event를 처리하는 thread(event loop)가 하나.
  - 단순하고, 이벤트 발생한 순서를 보장하게 만듬
  - 멀티 코어 cpu를 효율적으로 사용하지 못함
- 다중 thread event loop
  - event를 처리하는 thread(event loop)가 여러개
  - event queue하나에 여러 event loop가 접근하므로 race condition 발생 가능
  - 실행 순서를 제어하기 힘듬

Netty의 event loop
---------------

[참고자료](https://velog.io/@dailylifecoding/netty-study-memo-event-model)

- channel: 읽기, 쓰기, 연결, 바인드등의 작업을 수행하거나 네트워크 연결을 하는 요소. 비동기로 동작.
- event는 channel에서 발생
- 여러 channel은 하나의 event loop에 등록됨(1:N관계)
- 각 event loop은 자기만의 event queue 소유. 다수의 루프들이 이벤트 큐를 공유하면 race condition문제가 발생하거나 이벤트 순서가 안맞는 문제가 발생.
- 채널에서 발생된 이벤트는 해당 이벤트 루프가 소유한 이벤트 큐에 들어가서 순서대로 처리된다. 하나의 스레드에서 실행된 이벤트는 항상 같은 스레드(이벤트 루프)에서 수행된다

- netty에서는 event loop가 blokcing되면 안된다 -> 해당 event queue에 등록된 제대로 처리가 안됨.

Netty의 쓰레드
-------------

- threading model: os, pl, framework, applcation의 맥락에서 thread의 관리(생성, 유지, 제거 등)에 대한 전체적인 측면을 정의하는 것.
- 자바 초창기의 멀티 쓰레딩 체계는 동시 작업을 실행하기 위해 항상 새로운 스레드를 만들었음. 이는 성능의 저하를 불러왔음
- 자바5에서부터는 Thread 캐싱과 재사용을 통해서 성능을 개선하는 **Thread pool**을 지원하는 **Excutuor API**가 도입됨

- 새로운 방식
  - 요청된 작업(Runnable을 구현한 것)을 실행하기 위해 pool의 가용 리스트에서 thread하나를 선택해 할당
  - 작업이 완료되면 pool로 thread가 반환되고 재사용 됨

- 쓰레드를 생성하고 삭제하는 비용은 줄어들었지만, context-switch 비용은 사라지지 않았기 때문에 쓰레드의 수가 증가하면 부하가 심각해진다
- 동시성의 문제가 발생할 수도 있음

Eventloop 인터페이스
------------

- 이벤트 루프: 연결의 수명 기간동안 발생하는 이벤트를 처리하는 작업을 실행하는 것

```java
//간단한 이벤트 루프 구조
while(!terminated){
  List<Runnable> readyEvents = blockUntilEventsReady();
  for (Runnable e : readyEvents){
    ev.run();
  }
}
```

- netty의 이벤트 루프는 java.util.concurrent에 기본을 두는 스레드 실행자를 제공
- io.netty.channel 패키지의 클래스인 Channel 이벤트와 인터페이스를 수행하기 위한 API 확장

- Eventloop은 변경되지 않는 Thread에서 동작하며, Runnable or Callable을 Eventloop의 구현체에 제공하면 실행 or 예약된다.
- CPU와 같은 리소스 활용에 최적화된 갯수 만큼 eventloop가 생성되며, 하나의 eventloop당 여러 Channel이 연결될 수 있다

- Netty 4에서 이벤트 처리는 Eventloop에 할당된 Thread에 의해 처리된다
- Netty 3에서는 인바운드 이벤트만 Eventloop에서 처리했고 outbound 이벤트는 다른 쓰레드에서 처리가 가능했다.
  - ChannelHandler에서 outbound 이벤트를 동기화하는 것이 복잡해지는 문제가 나타났다. 왜냐하면 여러 쓰레드가 동시에 outbound event에 접근하는 문제가 발생할 수 있기 때문이다(동시에 Channel.write()하면 문제 발생)
  - 위의 이유로 Netty 4에서부터는 Eventloop가 모든 이벤트를 동일한 쓰레드에서 처리하도록 하였다

작업 스케쥴
-----------

- 작업을 나중에 실행하거나 주기적으로 실행해야할 떄가 있음
- 자바5 이전에는 작업스케쥴링을 표준 스레드와 동일한 한계를 갖는 백그라운드 thread를 이용하는 java.util.Timer를 통해 제공
  - newScheduledThreadPool, newSingleThreadScheduledExecutor 이용
- 위의 구현체들은 pool 관리 작업의 일부로 thread가 추가로 생성되는 한계점이 존재한다. 그래서 많은 작업을 예약하면 병목현상이 발생할 수 있다.
  - Netty는 Eventloop을 이용해서 이 문제를 해결

Netty 쓰레드 모델 세부 기능
----------------------

- Thread가 현재 Channel과 해당 Eventloop에 할당된 것인지를 확인하는 기능을 제공. Eventloop은 Channel 하나의 모든 event를 처리한다
- 호출 Thread가 Eventloop에 속하는 경우 해당 코드 블록이 실행되지만 그렇지 않으면 Eventloop가 나중에 실행하기 위해 작업을 예약하고 queue에 넣는다
- 이 방식은 Thread가 ChannelHandler와 동기화하지 않고도 Channel과 직접 상호작욜 할 수 있다

- - -

async Spring MVC vs Spring webflux
===============================

- [Servlet vs Reactive](https://www.infoq.com/news/2017/12/servlet-reactive-stack/)
- [참고자료](https://stackoverflow.com/questions/46606246/spring-mvc-async-vs-spring-webflux)

- Servlet async는 container와 request 처리 사이에 async를 제공
- Servlet async는 IO가 thread를 block함(container 단에서만 async를 제공하기 때문)
- Sevlet APi는 http request당 하나의 thread를 필요로 함
- WebFlux에서는 event loop를 사용하기 때문에 concurrency를 제공
