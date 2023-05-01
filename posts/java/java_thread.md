---
layout: post
title:  "Thread"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_thread.md
---

쓰레드
=======

프로세스 : 실행 중이 프로그램. 데이터, 메모리, 작업 쓰레드 로 구성

멀티쓰레드 : 하나의 프로세스내에서 여러 쓰레드가 동시에 작업을 수행하는 것. CPU의 코어 개수만큼 동작이 실행 가능하므로 실제 작업의 개수는 코어의 개수

멀티쓰레딩의 장점
  - CPU 사용률을 높임
  - 자원을 보다 효율적으로 사용
  - 사용자에 대한 응답성 향상
  - 작업이 분리되어 코드가 간결
  - IPC(system call을 통한 block이 됨)을 안해도 됨

여러 사용자에게 서비스를 제공하는 서버 프로그램의 경우 멀티 쓰레드로 작성해야 함. 만약 싱글쓰레드로 구성하게 되면 모든 요청마다 프로세스가 동작하게 되고 멀티쓰레드보다 더 시간과 메모리 소모가 커지게 됨.

멀티쓰레드의 단점은 **교착상태(deadlock), synchronization(동기화)** 가 있음

경우에 따라 **context swtiching**때문에 멀티쓰레드가 더 오래 걸리기도 한다

자바 쓰레드 모델
------------

#### green thread model

어플리케이션 level의 스레드가 user thread로 실행되고 jvm에서 하나의 스레드로 실행된다.
many(user thread)-to-one(kernel thread)형식이다. 멀티코어 위에서 돌아가더라도 하나의 jvm에서 실행되기 때문에 하나의 user thread만 실행된다. jvm에서 동작하는 하나의 thread가 blocking되면 모든 thread가 block된다.

스레드의 생명주기를 JVM에서만 관리하기 때문에 관리하는 측면에서 오버헤드가 줄어든다. context switching도 마찬가지이다.

#### native thread model

native thread model에서는 JVM이 os을 이용하여 kernel thread와 user thread를 매핑해서 사용한다. 기본적으로 many-to-many 모델이다. JVM은 application에서 생성한 *Thread* object(Thread package에서 제공)를 하나의 JNI를 이용하여 생성한 kernel thread(리눅스에서의 LWP)에서 실행되게 만든다.

이때, OS가 Linux이면 thread는 LWP(light weight process)이다. 리눅스에는 진정한 thread라는 개념이 없기 때문에 모든 스레드를 process와 같이 관리하고 스케쥴링한다. Process와 LWP의 차이는 LWP는 같은 process내이면 메모리 등을 공유한다.

- - -

구현
===

2가지 방법
  - **Thread** class 상속
  - **Runnable** interface 구현(자주 쓰임)

두 방법 모두 **run** 메서드를 구현해야 함

Runnable interface : 오직 **run** 이 정의되어 잇는 인터페이스

```java
//Thread 클래스 이용하여 생성
ThreadEx t1 = new ThreadEx();

//Runnable interface 이용
Thread t1 = new Thread(new ThreadEx());
```

Runnable interface를 구현하면 Thread class의 static method인 **currentThread()** 를 호출하여 쓰레드에 대한 참조를 얻어와야만 가능

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

**start()** 메서드를 호출하여 실행. cpu 스케줄러에 의해 start된 쓰레드가 run을 호출.

바로 실행되는 것이 아닌 실행 대기(wait,runnable)상태가 됨.

한 번 종료된 쓰레드는 다시 start할 수 없음. 새로운 thread를 실행해야 함

```java
t1.start();
t2.start();
```

main메서드 내에서 어떤 메서드의 **run** 메서드를 호출하게 되면 쓰레드를 생성하는 것이 아닌 그 메서드를 호출하는 것이 된다.

반면에 **start** 를 호출하게 되면, 새로운 **call stack** 을 생성하고 그 안에서 **run** 을 호출한다. 이후에 쓰레드가 종료되면 작업에 사용된 call stack이 소멸된다

main메서드 자체도 쓰레드이다. 싱글쓰레드 프로그램의 경우 프로그램이 종료되는 것은 main메서드가 종료되는 것을 의미했지만, 멀티쓰레드의 경우에는 main메서드가 종료되어도 다른 메서드가 종료되지 않아서 프로그램이 종료되지 않을 수 있다.

- - -

데몬 쓰레드
=========

데몬쓰레드는 일반 쓰레드를 돕는 보조적인 역할의 쓰레드. 일반 쓰레드가 모두 종료시 데몬 쓰데르도 강제로 종료된다. ex) grabage collector, auto save. display refresh

데몬 쓰레드는 무한루프와 조건문을 이용하여 실행 후 대기하고 있다가 특정 조건이 만족하면 다시 작업을 수행하도록 작성


```java
class MyThread implements Runnable{
  static boolean autoSave = false;

  public static void main(){
    Thread t = new Thread(new MyThread());
    t.setDaemon(true);
    t.start();

    for(int i=0;i<10;++i){
      try{
        Thread.sleep(1000);
      } catch(InterruptException e) {
        System.out.println(i);
        if(i==5)autoSave=true;
      }
    }
  }

  public void run(){
    while(true){
      try{
        Thread.sleep(3 * 1000);
      } catch(InterruptException e){
        if(autoSave){
          autoSave();
        }
      }
    }
  }

  public void autoSave(){
    System.out.println("AUTO SAVE!!!");
  }
}
```

- - -

우선순위
=========

쓰레드는 **priority** 라는 멤버변수를 이용하여 그 값에 따라 작업시간을 다르게 부여한다. 최대는 10 최소는 1이다.

```java
class MyClient{
  public static void main(String[] args){
    MyThread1 t1 = new MyThread1();
    MyThread2 t2 = new MyThread2();

    System.out.println(t1.getPriority());//5
    t2.setPriority(8);
    t1.start();
    t2.start();
  }
}

class MyThread1 extends Thread{
  public void run(){
    for(int i=0;i<300;++i){
      System.out.print("1");
      for(int j=0;j<1000000;++j);
    }
  }
}
class MyThread2 extends Thread{
  public void run(){
    for(int i=0;i<300;++i){
      System.out.print("2");
      for(int j=0;j<1000000;++j);
    }
  }
}
```

사실 멀티코어의 경우에는 둘의 차이가 거의 없을 수도 있다.

- - -

실행제어
========

쓰레드 프로그램이 어려운 이유는 **동기화, 스케쥴링** 때문이다.

스케쥴링을 위해서는 상태와 관련 메서드를 잘 이용해야 한다.

쓰레드는 다음과 같은 상태를 가진다
  - NEW : 쓰레드가 생성된 상태. start호출 전
  - RUNNABLE : 실행 중 혹은 실행 가능한 상태
  - BLOCKED : 동기화때문에 일시정지된 상태. lock이 풀릴 때 까지 기다린다
  - WAITING, TIMED_WAITING : 쓰레드의 작업이 종료되지 않고 일시정지된 상태.
  - TERMINATED : 종료된 상태

쓰레드의 상태변화
  - 쓰레드를 생성하면 **NEW** 상태이다. 이 후에 **start** 메서드가 호출되면, **RUNNABLE** 이 되고, 실행대기 **queue** 에 들어간다.
  - 자신의 차례가 되면 실행이 되다가 주어진 실행시간이 다 지나거나 **yield** 메서드가 호출되면, 실행이 중지되고 다시 queue에 들어간다
  - 실행 중 **suspend, sleep, wait, join, IO block** 등의 이유로 일시 정지되는 경우에는 **WAITING, BLOCKED** 상태가 되고 다시 **RUNNABLE** 상태가 될때까지 기다린다.
  - 지정된 일시정지시간(time-out)이 지나거나, **notify, resume, interrupt** 등의 메서드가 호출되면 다시 queue에 들어간다
  - 주어진 작업을 모두 실행을 마치거나 **stop** 메서드가 호출되면 종료된다(**TERMINATED**)

sleep
---------

일정 시간동안 쓰레드를 멈추게 한다.

해당 쓰레드를 호출한 메서드가 아닌 해당 쓰레드 내부에서 static 메서드로 호출해야한다.

```java
class Client{
  public static void main(String[] args){
    MyThread t1 = new MyThread();
  }
}

class MyThread extends Thread{
  public void run(){
    for(int i=0;i<300;++i){
      System.out.println(i);
      Thread.sleep(2000);
    }
  }
}
```

interrupt
----------

[참고자료](https://javafreak.tistory.com/210)

interrupt는 thread에게 신호를 보내는 것이다. 무시하고 말고는 해당 쓰레드 역할

```java
public class ThreaInterrupt implements Runnable{
    static private StringBuffer buf = new StringBuffer();
    static long start  = System.currentTimeMillis();
    static String lastMassage = "";

    public void run(){
        long end = System.currentTimeMillis();
        String message = "";
        while ( true ) {
            Thread.yield();

            if ( Thread.currentThread().isInterrupted())
                message = "Yielder : interrupted. ";

            if ( end - start > 1500){
                message += "Yielder : time over ";
                break;
            }
            else message += "Yielder : keep going!!! ";
            end = System.currentTimeMillis();
            print(message);
            message = "";
        }
        print(" : Yielder ends");
    }

    static void print(String msg){
        synchronized (buf) {
            if ( lastMassage.equals(msg)) return;       
            buf.append((System.currentTimeMillis() - start ) + " " + msg + "\n");
            lastMassage = msg;
        }
    }
    public static void main(String[] args) {
        // on the main thread stack

        Thread yielder = new Thread(new ThreaInterrupt());
        yielder.start(); // main thread makes yielder thread begin
        try {
            Thread.sleep(500);
            yielder.interrupt();
            print(">>>>>>>>>>>> first interrupt! ");
        } catch (InterruptedException e) {}

        try {
            Thread.sleep(1000);
            yielder.interrupt();
            print(">>>>>>>>>>>> second interrupt! ");
        } catch (InterruptedException e) {}
        print("Main Thread ends");

        try {    yielder.join(); } catch (InterruptedException e) {}

        System.out.println(buf.toString());
    }
  }
```

실행 중인 상태에서 **interrupt** 메서드가 호출되면, 그 쓰레드는 **isInterrupted** 로 확인해서 스스로 상응하는 작업을 수행해야 한다.

하지만, 쓰레드가 **sleep, wait, join** 에 의해 **WAITING** 상태에 있을 경우에는 **InterruptException** 이 발생하고 즉시 **RUNNABLE** 상태가 된다. 이 때 **isInterrupted** 는 false를 반환한다


suspend, resume, stop
---------------------

suspend : sleep처럼 쓰레드를 멈춘다.

stop : 강제 종료

resume : 정지된 쓰레드 실행대기 상태로 만든다

suspend와 stop은 교착 상태를 일으킬 수 있기 때문에 사용이 권장되지 않는다.

yield
----------

쓰레드 자신에게 주어진 실행시간을 다음 차례읭 쓰레드에게 **양보** 하고 실행대기 상태가 된다.

```java
class Client{
  public static void main(String[] args){
    MyThread t1 = new MyThread("1");
    MyThread t2 = new MyThread("2");

    t1.start();
    t2.start();

    try{
      Thread.sleep(2000);//2초간 기다린다
      t1.suspend();//t1을 suspend한다
      Thread.sleep(2000);
      t2.suspend();//t2을 suspend한다
      Thread.sleep(3000);
      t1.resume();//t1을 재개한다.
      Thread.sleep(3000);
      t1.stop();
      t2.stop();
    } catch(InterruptException e) {}
  }
}

class MyThread implements Runnable{
  private boolean suspended = false;
  private boolean stopped = false;

  private Thread thisTh;

  MyThread(String val){
    thisTh = new Thread(this, val);
  }

  public void run(){
    String name = th.getName();

    while(!stopped){
      if(!suspended){
        try{
          Thread.sleep(1000);
        } catch(InterruptException e){
          System.out.println("Interrupted");
        }
      } else {
        Thread.yield();
      }
    }
  }

  public void suspend(){
    suspended = true;
    thisTh.interrupt();
  }

  public void stop(){
    stopped = true;
    thisTh.interrupt();
  }

  public void resume(){
    suspended = false;
  }
  public void start(){
    th.start();
  }
}
```

**yield** 가 없으면 **suspended** 가 true이기 때문에 **while(!stopped)** 에 의해 while문을 아무 이유 없이 계속 반복하는 **busy waiting** 상태가 된다.

이 쓰레드가 정지 상태에 있을 경우 그 정지상태를 깨우기 위해 **interrupt** 를 메서드에 추가하였다.

join
----------

자신이 하던 작업을 잠시 멈추고 다른 쓰레드가 지정된 시간동안 작업을 수행하도록 기다린다. 시간을 지정하지 않으면 해당 쓰레드가 작업을 모두 마칠 때까지 기다린다.

join도 sleep 처럼 interrupt에 의해 대기 상태에서 벗어날 수도 있다.

```java
class Cliend{
  public void main(String[] args){
    MyThread t1 = new MyThread();

    t1.start();

    try{
      t1.join();//main thread가 t1이 종료될 때 까지 기다린다
    } catch(InterruptException e){
      //main 쓰레드를 interrupt할 경우
    }
  }
}
```

- - -

동기화
=======

멀티쓰레드는 여러 쓰레드가 같은 프로세스의 자원을 공유하기 때문에 서로의 작업에 영향을 줄 수 있다.

한 쓰레드가 **특정 지역(critical section)** 을 다른 쓰레드가 사용하지 못하도록록 **lock** 을 설정한다. lock을 건 쓰레드가 critical section의 작업을 전부 수행하면 lock을 해제한다. 그 후에 다른 쓰레드가 해당 지역에 접근할 수 있다.

이렇게 한 쓰레드가 작업 중인 부분을 다른 쓰레드가 간섭하지 못하도록 막는 것을 **동기화(synchronization)** 이라고 한다.

sychronized
-----------

가장 간단한 방법

```java
public synchronized void func(){
  //메서드 전체를 critical section으로 설정
}

public void func2(){
  synchronized(this){
    //특정 부분만 critical section 설정
  }
}
```

다음과 같은 코드는 critical section에 private 멤버변수가 존재하기 때문에 문제가 발생할 수 있다. synchronized는 단순히 지정된 영역을 보호하는 것이기 때문이다.

```java
class A{
  public int balance = 1000;

  public synchronized void withdraw(int money){
    if(balance >= money){
      try{
        Thread.sleep(1000);
      } catch(InterrupException e){}
      balance -= money;
    }  
  }
}
```

wait, notify
------------

작업이 제대로 진행되지 않을 때 lock을 건 상태로 오래동안 있지 않도록 하는 것이 중요하다. 이럴 경우를 위해서 **wait** 으로 해당 lock을 반납하고 기다리다가, **notify** 를 통해서 lock을 다시 얻고 작업을 수행할 수 있다

**wait** 메서드를 호출하면 쓰레드가 **waiting pool** 에 들어가게 된다. 이 후에 **notify** 가 호출되면 대기 중인 임의의 쓰레드에게 lock을 설정할 수 있다고 통보한다. **notifyAll** 은 waiting pool의 모든 쓰레드가 통보를 받는다.

**waiting pool** 은 공유 객체마다 존재하기 때문에 **notifyAll** 을 하더라도 해당 공유 객체를 사용하는 쓰레드만 적용된다.

이 메서드들은 모두 **Object 클래스** 안에 정의되어 있기 때문에 모든 객체에서 호출 가능하고, **synchronized** 블록 안에서만 사용가능하다.

```java
public class WaitNotifyExample {
    public static void main(String[] args) {
        Work sharedObject = new Work();

        ThreadA threadA = new ThreadA(sharedObject);
        ThreadB threadB = new ThreadB(sharedObject);

        threadA.start();
        threadB.start();
    }
}

public class Work{
    public synchronized void methodA() {
        notify(); // 일시정지 상태에 있는 ThreadB를 실행 대기상태로 만듬
        try {
            wait(); // ThreadA를 일시 정지 상태로 만듬
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void methodB() {
        notify(); // 일시정지 상태에 있는 ThreadA를 실행 대기상태로 만듬
        try {
            wait(); // ThreadB를 일시 정지 상태로 만듬
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
public class ThreadA extends Thread{
    private WorkObject workObject;

    ThreadA(WorkObject workObject) {
        this.workObject = workObject;
    }

    public void run() {
        for(int i=0; i<10; i++) {
            workObject.methodA(); // 공유객체의 methodA를 반복적으로 호출
        }
    }
}

public class ThreadB extends Thread{
    private WorkObject workObject;

    ThreadB(WorkObject workObject) {
        this.workObject = workObject;
    }

    public void run() {
        for(int i=0; i<10; i++) {
            workObject.methodB(); // 공유객체의 methodA를 반복적으로 호출
        }
    }
}
//출처 : https://cornswrold.tistory.com/189
```

위의 코드의 경우 2개의 쓰레드기 때문에 문제가 없지만, 해당 객체에 접근하는 쓰레드의 수가 많으면 특정 쓰레드가 notify를 받지 못해서 오래도록 기다리는 **starvation** 문제가 발생할 수 있다. 이를 해결하기 위해서는 **notify** 보다는 **notifyAll** 을 사용해야 한다.

하지만 **notifyAll** starvation은 해결할 수 있지만, 여러 쓰레드가 lock을 얻기 위해서 **race condition** 에 돌입할 수 있다.

Lock, Condition
------------------

**synchronized** 는 편리하게 사용할 수 있지만, 필요에 따라 lock를 다양하게 설정해서 써야할 때가 있다. 이럴 때 **lock 클래스** 를 이용하면 된다.

lock 클래스 종류
  - ReentrantLock : 재진입이 가능한 lock. 일반적인 배타 lock
  - ReentrantReadWriteLock : 읽기에는 공유적이지만, 쓰기에는 배타적인 lock. 해당 지역에 읽기 lock이 걸려있으면 다른 쓰레드가 읽기 lock을 걸 수 있다
  - StampedLock : ReentrantReadWriteLock에 낙관적인 lock 기능 추가. 무조건 read lock을 걸지 않고, read와 write가 충돌할 때만 write가 끝난 후에 read lock을 건다.

ReentrantLock의 경우에는 **ReentrantLock(boolean fair)** 와 같이 **공정한** lock 분배를 할 수 있게 생성자를 호출할 수 있다. **fair==true** 인 경우에는 가장 오래 기다린 쓰레드가 lock을 걸 수 있다. 그러나 기다린 정도를 계산하기 위해 다른 작업이 필요하고 성능이 떨어질 수 있다.


lock을 사용하는 방법은 다음과 같다

```java
//기존의 synchrnoized
synchronized(lock){
  //critical section
}

//ReentrantLock 사용
private ReentrantLock lock = new ReentrantLock();
lock.lock();
try{
//critical section
} finally {
  lock.unlock();
}

System.out.println(lock.isLocked());
```

응답성이 중요하여 일정 시간정도만 lock을 기다리도록 하기 위해서는 **tryLock** 을 사용한다

```java
boolean tryLock(long timeOut, TimeUnit unit) throws InterruptException;
```

**notify, notifyAll** 은 특정 조건에 맞게 쓰레드를 선택하지 않아서 **race condition, starvation** 등의 문제가 생길 수 있다. 이럴 때 **Condition** 을 사용한다.(완벽하게 해결해주지는 않는다)

- await, awaitUninterruptibly, awaitNanos, awaitUntil : wait 역할
- signal, signallAll : notify, notifyAll 역할

다음의 케이스를 보자

```java
public class ConditionExample {
    public static void main(String[] args) {
        Work sharedObject = new Work();

        ThreadA threadA = new ThreadA(sharedObject);
        ThreadB threadB = new ThreadB(sharedObject);

        threadA.start();
        threadB.start();
    }
}

public class Work{
  private boolean food = false;
  public synchronized void methodA() {
    while(food){
      try{
        wait();//methodA must wait
      } catch(InterruptException e) {}
    }
    numFood = true;
    notify();
  }

  public synchronized void methodB() {
    while(!food){
      try{
        wait();//methodA must wait
      } catch(InterruptException e) {}
    }
    numFood = false;
    notify();
  }
}
public class ThreadA extends Thread{
  private WorkObject workObject;

  ThreadA(WorkObject workObject) {
    this.workObject = workObject;
  }

  public void run() {
    for(int i=0; i<10; i++) {
        workObject.methodA(); // 공유객체의 methodA를 반복적으로 호출
    }
  }
}

public class ThreadB extends Thread{
    private WorkObject workObject;

    ThreadB(WorkObject workObject) {
        this.workObject = workObject;
    }

    public void run() {
        for(int i=0; i<10; i++) {
            workObject.methodB(); // 공유객체의 methodA를 반복적으로 호출
        }
    }
}
```

Food가 없는 경우(false) ThreadA가 작업을 수행해야 한다. 하지만 notify를 통해서 지속해서 ThreadB가 불려지게 되면 starvation현상이 발생할 수 있다.

이 현상을 위의 **Condition** 을 이용하여 해결해보자

```java
public class ConditionExample {
    public static void main(String[] args) {
        Work sharedObject = new Work();

        ThreadA threadA = new ThreadA(sharedObject);
        ThreadB threadB = new ThreadB(sharedObject);

        threadA.start();
        threadB.start();
    }
}

public class Work{
  private boolean food = false;
  private Reentrant lock = new ReentrantLock();
  private Condition conditionA = lock.newCondition();
  private Condition conditionB = lock.newCondition();
  public void methodA() {
    lock.lock();  
    while(food){
      try{
        conditionA.await();//methodA must wait
      } catch(InterruptException e) {}
    }
    numFood = true;
    conditionaB.signal();
    lock.unlock();
  }

  public void methodB() {
    lock.lock();  
    while(!food){
      try{
        conditionB.wait();//methodA must wait
      } catch(InterruptException e) {}
    }
    numFood = false;
    conditionA.signal();
    lock.unlock();
  }
}
public class ThreadA extends Thread{
  private WorkObject workObject;

  ThreadA(WorkObject workObject) {
    this.workObject = workObject;
  }

  public void run() {
    for(int i=0; i<10; i++) {
        workObject.methodA(); // 공유객체의 methodA를 반복적으로 호출
    }
  }
}

public class ThreadB extends Thread{
    private WorkObject workObject;

    ThreadB(WorkObject workObject) {
        this.workObject = workObject;
    }

    public void run() {
        for(int i=0; i<10; i++) {
            workObject.methodB(); // 공유객체의 methodA를 반복적으로 호출
        }
    }
}
```

volatile
-----------

프로세서(CPU)는 각 CPU마다 cache라는 저장소가 존재한다. memory의 데이터에 접근하기 전에 cache에 접근해서 있는지 확인을 먼저 한다.

멀티쓰레드로 작업을 수행할 때, 각 코어마다 cache의 값이 달라지게 되기도 한다. **synchrnoized** 블록 안에서 해당 변수 값을 변경하고 블록에서 나올 경우에 캐시 메모리간의 불일치 문제가 발생할 수 있다, 이 때 **volatile** 을 사용하여 해당 변수를 원자화할 수 있다.

```java
volatile int balance = 0;
volatile int MAX = 10;

public synchronized void add(int money){
  if(balance + money < MAX){
    System.out.println("입금 가능");
    balance += money;
  }
}

public void withdraw(int money){
  if(balance >= money) {
    System.out.println("출금 가능");
    balance -= money;
  }
}
```

위의 코드의 경우 **synchronized** 를 하지 않았기 때문에 **add** 와 **withdraw** 가 동시에 호출이 될 수 있고, **balance**  가 원자화 되어 있다고 해도 문제가 발생할 수 있다.

또는, 8byte인 long과 double 멤버변수를 여러 쓰레드가 동시에 접근하는 경우(해당 변수들은 한번에 읽거나 쓰지 않는다)에 생길 수 있는 문제를 **volatile** 로 해결할 수 있다.

- - -

쓰레드 풀
===========

쓰레드를 만들고 쓰고 다시 반납하는 것도 결국에는 자원을 소모하게 된다. 일반적으로 지속적으로 발생하는 병렬작업을 위해 생성, 삭제를 하는 것은 성능을 저하시키고. 작업의 숫자를 제한해서 안정적인 운영을 가능하게 한다. 물론 가끔 놀고 있는 쓰레드들이 만들어진다.

**Thread pool** 을 만들어 놓고 쓰레드를 제한된 개수만큼 생성하고 **Queue** 에 들어오는 작업들을 pool의 thread들에 배분하는 것이다.

생성
---

쓰레드 풀을 생성하기 위해서는 **ExecutorService** interface와 **Executor** 클래스를 사용한다. ExecutorService 구현 객체는 Executor클래스 메서드를 통해 생성할 수 있다

```java
//초기:0, 코어쓰레드(최소한 유지) : 0, 최대 : Integer.MAX_VALUE
//60초간 운영되지 않는 쓰레드는 삭제
ExecutorService es1 = Executors.newCachedThreadPool();

//초기:0, 코어쓰레드(최소한 유지) : nThreads, 최대 : nThreads
ExecutorService es2 = Executors.newFixedThreadPool(nThreads);
```

종료
---

쓰레드 풀의 쓰레드들은 데몬 쓰레드가 아니라서 main쓰레드가 종료되어도 계속 남아 있는다. 강제로 종료하기 위해서 다음의 메서드들을 사용한다.

```java
//남아있는 작업을 모두 완료한 후 종료
es1.shutdown();

//남아있는 작업을 무시하고 모두 종료
es1.shutdownNow();

//timeout동안 기다리고 작업이 완료되지 않으면 그 작업스레드를 interrupt하고 false를 반환
es1.awaitTermination(long timeout, TimeUnit unit);
```

작업 생성
-------

작업은 **Runnable, Callable** 이 있고, 둘의 차이는 반환값의 존재여부다.

```java
Runnable runnableTask = new Runnable(){
  @Override
  public void run(){
    //
  }
}

Callable callableTask = new Callable<T>(){
  @Override
  public T call(){
    return T;
  }
}
```

작업 처리 요청
-----------

작업을 처리하기 위해서 생성된 작업을 작업 queue에 넣어야 한다.

```java
//Runnable을 큐에 저장
//작업 결과를 받지 못함
//예외 발생시 쓰레드 제거
s1.execute(runnableTask);

//Runnable또는 Callable을 큐에 저장
//리턴된 Future을 통해 작업처리 결과를 얻을 수 있다.
//예외 발생시 쓰레드 제거하지 않고 재사용
Future<> res = s1.submit(runnableTask);//res.get()이 null 반환
Future<> res = s1.submit(runnableTask, result);//res.get()이 result의 타입 반환
Future<> res = s1.submit(callableTask, result);//res.get()이 callabe의 결과 반환
```

**Future** 은 작업결과가 아니라 작업을 기다리는 데에 사용(pending completion 객체). 다음의 방법들을 이용하여 작업이 완료될 때까지 block하다가 결과를 얻는다.

```java
//완료될 때까지 기다린다
res.get();

//일정 시간 기다린다.
//완료되지 않으면 TimeoutException발생
res.get(1000,TimeUnit.millies);
```

작업 종료
--------

작업을 강제로 종료하거나 취소여부를 확인하기 위해 다음 메서드를 사용한다

```java
//작업취소. Running을 interrupt할지 말지 선택 가능
res.cancel(mayInterruptRunning);

//취소 여부 확인
res.isCancelled();

//작업 종료 여부 확인
res.isDone();
```


아래는 위의 내용들을 적용한 예제이다.

```java
class Client{
  public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();

    class Task implements Runnable {
        Result result;
        Task(Result result) {
            this.result = result;
        }

        @Override
        public void run() {
            int sum = 0;

            for (int i = 1; i <= 10; i++) {
                sum += i;
            }
            result.addValue(sum);
        }
    };

    Result result = new Result();
    Runnable task1 = new Task(result);
    Runnable task2 = new Task(result);
    Future<Result> future1 = executorService.submit(task1);
    Future<Result> future2 = executorService.submit(task2);

    try {
        future1.get();
        future2.get();
        System.out.println(result);
    } catch (Exception e) {
        e.printStackTrace();
    }

    executorService.shutdown();
  }
}

class Result {
    int accumValue = 0;
    public synchronized void addValue(int value) {
        accumValue += value;
    }
}
```

CompletionService
--------------------

모든 작업을 요청 순서대로 받는 것이 아니기 때문에 먼저 종료된 결과부터 받을 수 있게 해준다.

```java
ExecutorService es = Executors.newCachedThreadPool();
CompletionService<String> cs = new ExecutorCompletionService<String>(es);

cs.submit(task1);
cs.submit(task2);

//완료된 작업이 있으면 해당 future 반환, 없으면 null반환
cs.poll();
//시간안에 완료된 작업이 있으면 반환 해당 future 반환, 없으면 timeout동안 blocking
cs.poll(timeout, TIMEUNIT);
//완료된 작업이 있으면 반환 해당 future 반환, 없으면 있을때까지 blocking
cs.take();

//완료된 결과가 있을때까지 반복해서 실행되는 쓰레드
es.submit(new Runnable() {
    public void run() {
        while (true) {
            try {
                Future<String> future = completionService.take();
                String result = future.get();
            } catch (Exception e) { break; }
        }
    }
});
```

callback
----------

**CompletionHandler** 를 사용하여 자바스크립트의 **callback** 과 같이 행동하는 작업을 생성할 수 있다.

```java
CompletionHandler<V, A> callback = new CompletionHandler<V, A>() {
    @Override
    public void completed(V result, A attachment) {
      //성공시
    }

    @Override
    public void failed(Throwable exc, A attachment) {
      //실패시
    }
}

Runnable task = new Runnable() {
    public void run() {
        try {
          //작업
          callback.completed(result, null);
        } catch (NumberFormatException e) {
          //실패
          callback.failed(e, null);
        }
    }
};
es.submit(task);
```


- - -

참고자료
======

- https://palpit.tistory.com/732
- https://medium.com/@unmeshvjoshi/how-java-thread-maps-to-os-thread-e280a9fb2e06
- https://medium.com/@leeyh0216/distributed-system-processes-1-9101d5c5ceee
