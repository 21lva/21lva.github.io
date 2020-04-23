---
layout: post
title:  "Thread"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_thread.md
---

- 멀티쓰레딩의 장점
  - CPU 사용률을 높임
  - 자원을 보다 효율적으로 사용
  - 사용자에 대한 응답성 향상
  - 작업이 분리되어 코드가 간결

구현
===

- 2가지 방법
  - **Thread** class 상속
  - **Runnable** interface 구현(자주 쓰임)
- 두 방법 모두 **run** 메서드를 구현해야 함

- Runnable interface : 오직 **run** 이 정의되어 잇는 인터페이스

```java
//Thread 클래스 이용하여 생성
ThreadEx t1 = new ThreadEx();

//Runnable interface 이용
Thread t1 = new Thread(new ThreadEx());
```

- Runnable interface를 구현하면 Thread class의 static method인 **currentThread()** 를 호출하여 쓰레드에 대한 참조를 얻어와야만 가능

```
//Runnable interface을 구현한 클래스에서 getName()하기
class MyThread implements Runnable{
  public void run(){
    System.out.println(Thread.currentThread().getName());//현재 thread 이름 반환
  }
}
```

실행
=====

- **start()** 메서드를 호출하여 실행
- cpu 스케줄러에 의해 start된 쓰레드가 run을 호출
- 바로 실행되는 것이 아닌 실행 대기(wait,runnable)상태가 됨
- 한 번 종료된 쓰레드는 다시 start할 수 없음. 새로운 thread를 실행해야 함


```java
t1.start();
t2.start();
```
