---
layout: post
title:  "Singleton"
date:   2020-06-02 01:02:59
author: Inhyuk
category: oop
cover:  "/assets/instacode.png"
name: singleton.md
---

예시
====

- 다음은 하나만 존재하는 프린터를 생성하는 코드이다

```java
public class Printer{
  private static Printer printer = null;
  private static int cnt;

  public static Printer getPrinter(){
    if(printer == null) {
      cnt = 0 ;
      printer = new Printer();
    }

    return printer;
  }

  public void print(String str){
    System.out.println((cnt++) + " : " + str);
  }
}
```

- 위의 코드는 객체 생성을 검사하는 **if(printer==null), cnt++** 이 thread safe하지 않다(race condition)

- 이 방법은 다음과 같이 2가지 방법으로 고칠 수 있다.
  - 정적 변수에 인스턴스 변수를 만들어 바로 초기화 : 정적 변수는 객체가 아닌 클래스가 메모리에 로드시 만들어지고 초기화 된다.
  - 인스턴스를 만드는 메서드를 동기화

- 두 번째 방법을 구현한 코드이다


```java
public class Printer{
  private static Printer printer = null;
  private static int cnt;

  public synchronized static Printer getPrinter(){
    if(printer == null) {
      cnt = 0 ;
      printer = new Printer();
    }

    return printer;
  }

  public void print(String str){
    synchronized(this){
      System.out.println((cnt++) + " : " + str);
    }
  }
}
```

- 객체 생성을 하는 메서드를 동기화 시켜서 한 쓰레드만 사용할 수 있게 했다.
- cnt값을 여러 쓰레드가 접근하는 것을 막기 위해 동기화 하였다.

- - -

Singleton Pattern
=============

- 인스턴스가 오직 하나만 생성되는 것을 보장하고 어디에서든 이 인스턴스에 접근할 수 있도록 하는 패턴
- 클라이언트가 싱글턴 클래스를 요청하면 객체가 존재하는지를 확인하고 생성하거나 전달한다.

vs Static class
-------------------

- static class는 클래스의 객체를 생성하지 않아도 되기 때문에 안전. 그러나 인터페이스를 구현할 수 없다.
- 단위 테스트, 구현의 변경등을 위해서 인터페이스를 사용하는 경우에는 싱글턴 패턴을 사용해야 한다.
