---
layout: post
title:  "코루틴"
date:   2022-07-10 02:02:59
author: Inhyuk
category: kotlin
tags: kotlin
cover:  "/assets/instacode.png"
name: coroutine.md
---

코루틴
====

- 실행 -> 일시 중단 -> 재시작이 가능한 비선점형 멀티태스킹 서브 루틴
  - 서브루틴: 함수, 메서드, 프로시저 등을 포괄하는 용어.
  - 비선점형 멀티태스킹: 선점(preemptive)는 다른 실행대상의 제어권을 뺐는 것이다. 비선점형 멀티태스킹은 cpu, memory와 같은 자원을 선점 없이 여러 프로세스가 동시에 사용할 수 있는 방법
- generator도 대표적인 코루틴
- 코루틴은 특정 객체 타입이 아니다. Deferred, Job 등 가능.
- 코루틴 내부에 다른 코루틴을 포함 가능하다.

vs 스레드
-------

```
import kotlin.concurrent.thread

class DemoApplication

fun main(args: Array<String>) {
    val thread1 = thread {
        println("Thread in")
        Thread.sleep(1000)
        println("Thread out")
    }

    println("Main")
    thread1.join()
}

//결과
Thread in
Main
Thread out
```

- 코루틴은 **light-weight thread**라고 불린다.
- 코루틴은 non-preemptive, thread는 preemptive.
- 스레드는 할당된 시간안에만 CPU에 대한 제어권을 갖지만, 코루틴은 같은 스레드내에서 다른 코루틴에게 제어를 건내주기 때문에 해당 스레드는 지속적으로 같은 CPU를 사용한다. OS가 아닌 application에서의 time-share가 이루어지는 것.
- 코루틴은 꼭 같은 스레드에서 실행해야하는 것은 아니다 .
- 코루틴에서는 context-switching이 발생하지 않는다.
- 코루틴은 스레드와 달리 자체 호출 스택을 갖고 있다. 코루틴에서 하위 서브루틴을 제어할 수 없다.
- 코루틴은 값을 반환할 수 있고, 예외를 사용할 수 있다.

```
fun main(args: Array<String>)  = runBlocking {
    val x = coroutineScope {
        println("Coroutine in")
        delay(2000)
        println("Coroutine out")
        "return value"
    }

    println("Main")
    println(x)
}

//result
Coroutine in
Coroutine out
Main
return value
```

- - -

코틀린과 코루틴
===========

- kotlin에서도 coroutine을 지원한다.
- kotlin에서의 코루틴은 언어가 지원하는 것이 아니라 coroutine을 구현할 수 있는 기본 도구를 kotlin에서 제공한다고 보면 된다(**kotlinx.coroutines, kotlin.coroutines**)

빌더
---

- kotlinx 패키지에는 코루틴을 만들 수 있는 다양한 coroutine builder들을 제공한다

#### launch

- coroutine을 job으로 반환하는 함수. 만들어진 즉시 실행.
- launch가 작동하려면 **CoroutineScope**객체가 블록의 *this* 이어야 함. 다른 suspend 함수 내부라면 그 함수의 **CoroutineScope**를 사용하면 되고, 그렇지 않으면 **GlobalScope**를 사용해야한다

```
fun main(args: Array<String>)  {
    GlobalScope.launch {
        println("Coroutine in")
        delay(2000)
        println("Coroutine out")
    }

    println("Main")
    Thread.sleep(5000)
}

//결과
Main
Coroutine in
Coroutine out
```

#### runBlocking

- GlobalScope는 main thread에서 실행되기 때문에 main thread가 종료되면 실행되지 않는다. launch가 모두 실행될때까지 기다리고 싶으면 코루틴이 종료될 때까지 현재 스레드를 blocking하는 *runBlocking()*을 이용해야한다
- runBlocking도 코루틴을 만들어주는 코루틴 빌더
- 코루틴 내에서 다른 코루틴이 생성되면 자식 코루틴들이 멈출 때까지 계속 반복되고, 부모 코루틴이 취소되면 자식 코루틴들도 recursive하게 취소된다.

```
fun main(args: Array<String>)  {
    runBlocking {
        launch {
            println("Coroutine in")
            delay(2000)
            println("Coroutine out")
        }
    }

    println("Main")
}

//result
Coroutine in
Coroutine out
Main
```

- *yield*를 사용하면 코루틴 중간에 suspend된다

```Kotlin
fun main(args: Array<String>)  {
    runBlocking {
        launch { //launch는 즉시 반환된다
            println("1-1")
            yield()
            println("1-2")
            delay(2000) //delay를 이용하면 해당 코루틴은 그 시간만큼 기다리다가 다시 예약된다
            println("1-3")
        }
        println("After first")
        launch {
            println("2-1")
            yield()
            println("2-2")
            yield()
            println("2-3")
        }
        println("After second")
    }

    println("Main")
}

//result
After first
After second
1-1
2-1
1-2
2-2
2-3
1-3
Main
```

#### async

- **launch**와 비슷하지만 launch와 달리 **Deferred**를 반환. Deferred는 Job을 상속한다
- Deferred는 Job과 달리 Generic이고, *await()* 함수가 정의되어 있다. await 함수는 반환값을 기다리는 용도로 사용된다.
- 따로 설정하지 않으면 launch처럼 코루틴을 반환 즉시 코루틴을 시작시킨다.

```
fun main(args: Array<String>)  {
    val startTime = System.currentTimeMillis()
    runBlocking {
        val a1 = async {
            delay(2000)
            println(Thread.currentThread())
            1
        }
        val a2 = async {
            delay(2000)
            println(Thread.currentThread())
            2
        }
        println(a1.await()+a2.await())
        println("Wait")
        val a3 = async {
            delay(2000)
            println(Thread.currentThread())
            3
        }

        println(a3.await())
    }
    println(System.currentTimeMillis()-startTime)

    println(Thread.currentThread())
}

//result
Thread[main,5,main]
Thread[main,5,main]
3
Wait
Thread[main,5,main]
3
4155
Thread[main,5,main]
```

- 위의 코드에서 알 수 있는 것은 2가지이다: 순차적으로 실행하면 6000 millisecond가 소요되어야 하지만 실제로는 그것보다 적게 소요되었다. 모든 코루틴이 같은 스레드에서 동작하였다. 즉, 코루틴과 스레드의 가장 큰 차이를 보여주는데, 여러 작업을 concurrent하게 수행하면서도 적은 스레드를 필요로 한다는 장점이다.
- async는 job이 생성되는 즉시 실행이 된다. 만약에 특정 시점에 시작을 하게 하고 싶으면 아래와 같이 한다

```
fun main(args: Array<String>)  {
    runBlocking {
        async {
            println("in1")
            delay(1000)
            println("in2")
        }
        delay(1000)
        println("out")
    }
    runBlocking {
        val j = async (start = CoroutineStart.LAZY){
            println("in1")
            delay(1000)
            println("in2")
        }
        delay(1000)
        println("out")
        j.start()
    }
}

// result
in1
out
in2
out
in1
in2
```

coroutine context
------------------

- CoroutineScope: 코루틴의 범위, 블록을 묶음으로 제어하는 단위
  - GlobalScope: CoroutineScope의 한 종류. Application의 생명 주기에 종속적
  - launch, async: CoroutineScope의 extension method. 다양한 요구사항에 맞게 **코루틴을 생성**
  - CoroutineScope는 CoroutineContext라는 field를 갖고 있다. launch, async에서 해당 field를 사용.
- CoroutineContext: job들과 dispatcher를 저장하는 일종의 Map. coroutine runtime의 dispatcher의 정보를 이용하여 다음 작업을 선정하고, 배정할 스레드를 선택.

#### Dispatcher

- CoroutineScope의 주요 요소.
- dispatcher에는 어떤 스레드에서 현재 coroutine을 실행할지에 대한 정보를 제공
  - Dispatchers.Unconfined: 현재 caller thread에서 시행. suspend 후에는 다른 적절한 thread에서 실행되기도 함.
  - Dispatchers.Default: DefaultDispatcher thread에서 실행. GlobalScope의 dispatcher사용
  - 미지정: 부모 코루틴의 context의 dispatcher 사용
- EmptyCoroutineContext: default로 제공되는 singleton context.

#### asContentElement

- coroutine은 여러 스레드에서 실행될 수 있기 때문에 *ThreadLocal*과 같은 기능을 쓸 수 없다.
- 대신해서 사용할 수 있는 방법이 있는데 1) context에 값 저장(plus, get, minusKey 등 이용), 2)ThreadLocal의 *asContentElement* 사용하는 방법이다

```
fun main(args: Array<String>)  {
    val threadLocal = ThreadLocal<String>()
    runBlocking {
        launch (newSingleThreadContext("new-thread-1")+threadLocal.asContextElement(value = "abc")){
            println(threadLocal.get())
            yield()
            println(threadLocal.get())
        }
    }
    val threadLocal2 = ThreadLocal<String>()
    threadLocal2.set("abcd")
    runBlocking {
        launch (newSingleThreadContext("new-thread-2")){
            println(threadLocal.get())
        }
    }
}

//result
abc
abc
null
```

suspend
--------

- 코루틴에서도 일반 함수만 사용하는 경우에는 코루틴 내에서는 순차적으로 업무를 수행한다.
- 코루틴에서 함수가 일시 중지 되지 않는다면 같은 Thread내에서 다른 코루틴을 수행하지 않는다.
- 함수 앞에 *suspend* 키워드를 붙이면 일시 중단 가능한 함수를 만들 수 있다.
- 코루틴 일시 중단은 코루틴 내부에서 이루어져야한다.

```kt
fun main(args: Array<String>)  {
    runBlocking {
        println("start")
        launch {
            f1()
        }
        launch {
            f1()
        }
        println("end")
    }

}

suspend fun f1(){
    println("1")
    yield()
    println("2")
    yield()
    println("3")
    yield()
    println("4")
}
```

```
//실행 결과
start
end
1
1
2
2
3
3
4
4

//yield를 제거한 경우의 실행결과
start
end
1
2
3
4
1
2
3
4
```

- 위의 예시에서 yield는 코루틴에 진입하거나 나올 때에 코루틴의 상태 및 위치를 저장해야하고, 다음에 어떤 코루틴이 실행 가능한지에 관한 기능이 필요하다. 전자는 kotlin의 컴파일러가 담당하고 후자는 컨택스트의 dispatcher에 의해 수행된다.
- kotlin의 CPS(continuation passing style) 변환과 state machine을 이용해서 코드를 생성.

cancelling
-----------

- coroutine의 *cacel()*을 시행하고 코루틴 내부에서 *yield()*를 시행하면 **CancellationException**이 발생한다. 혹은, *isActive*로 상태를 체크할 수 있다.

```
fun main(args: Array<String>)  {
    runBlocking {
        val c = launch { f1() }
        println("wait")
        delay(4000)
        println("cancel")
        c.cancel()
    }
}

suspend fun f1(){
    while(true){
        println("1")
        yield()
        delay(1000)
    }
}

//result
wait
1
1
1
1
cancel
```

- 만약에 취소하더라도 꼭 다뤄야 하는 작업이 있거나, 취소를 로깅하려면 *try-catch-finally*를 사용하면 된다

```
fun main(args: Array<String>)  {
    runBlocking {
        val c = launch { f1() }
        println("wait")
        delay(4000)
        println("cancel")
        c.cancel()
    }
}

suspend fun f1(){
    try {
        while(true){
            println("1")
            yield()
            delay(1000)
        }
    } catch (c: CancellationException){
        println("Cancelled")
    } finally {
        println("finally")
    }
}

// result
wait
1
1
1
1
cancel
Cancelled
finally
```

- 일정 시간 이후에 job을 취소하고 싶으면 *withTimeout*을 사용

```
fun main(args: Array<String>)  {
    runBlocking {
        try {
            withTimeout(4000L){
                while(true){
                    println("1")
                    delay(1000)
                    yield()
                }
            }
        } catch (c: TimeoutCancellationException){
          //withTimeout은 TimeoutCancellationException을 발생시킨다
            println("cancelled")
        }
    }
}

//result
1
1
1
1
cancelled
```

Exception
-------------

- coroutine은 내부에서 일어나는 Exception을 2가지 방법으로 다룬다
  1. 외부로 propagation: Exception 발생 즉시 외부로 Exception전달. ex) launch, actor
  2. expose: Exception 발생 즉시 외부로 전달하지는 않고 특정 상황(method를 호출한다던지)에서 Exception 전달. ex) async, produce

```kt
//propagation
fun main(args: Array<String>)  = runBlocking {
    val co = GlobalScope.launch {
        throw IndexOutOfBoundsException()
    }
    delay(1000)
    println("Cannot get")
}

//expose
fun main(args: Array<String>)  = runBlocking {
    val co = GlobalScope.async {
        throw IndexOutOfBoundsException()
    }
    delay(1000)
    println("Can get")
    co.await()
    println("Cannot get")
}
```

- **CoroutineExceptionHandler**: coroutine 내부에서 propagation되는 에러를 핸들링할 때 사용. async와 같은 경우에는 사용 불가

```kt
fun main(args: Array<String>)  = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println(exception.message)
    }
    val co = GlobalScope.launch(handler){
        throw IndexOutOfBoundsException("Error from Launch")
    }
    val co2 = GlobalScope.async(handler){
        throw IndexOutOfBoundsException("Error from Async")
    }
    println("Can get")
    co2.await()
    println("Cannot get")
}

//결과
Can get
Error from Launch
Exception in thread "main" java.lang.IndexOutOfBoundsException: Error from Async
```

- 위에서 알아본 **CancellationException**은 Exception을 catch하지 않아도 시스템이 다운되지는 않는다. try-catch로 취소된 상황에 대처하기 위한 용도. ex) resource release 등
- 그 외의 Exception이 발생하면 부모 코루틴까지 전부 취소시키고 위로 Exception을 전파한다

```kt
fun main(args: Array<String>)  = runBlocking {
    val co2 = GlobalScope.async{
        println("inner")
        delay(10000)
    }
    co2.cancel()
    println("Cancelled")
}

//결과
inner
Cancelled
```

Channel
---------

- async로 생성된 *Deferred*는 하나의 값을 return하지만, **Channel**은 stream을 return.
- 서로 네트워킹 해야하는 두 코루틴 사이에 정보를 교환하기 위한 용도로 사용된다

```kt
fun main(args: Array<String>) {
    runBlocking {
        val ch = Channel<Int>()
        val co1 = launch {
            for(i in 1..10){
                ch.send(i)
                delay(1000)
            }
            ch.close()
        }
        val co2 = launch {
            println(ch.receive()) //receive()는 받아올 값이 없으면 suspend된다
            for(v in ch){ //iterator가 close되어있을때까지 전송
                println(v)
            }
        }
    }
}

//result
1
2
3
4
5
6
7
8
9
10
```

- *produce*라는 coroutine builder를 이용하면 생산자 channel를 쉽게 만들 수 있다. 이 producer에 해당하는 consumer는 *consumeEach*로 만들 수 있다. consumeEach을 쓸때 여러 코루틴에서 consume을 한다면 하나의 컨슈머에서 문제가 생기면 channel이 닫히면서 다른 코루틴에도 영향을 미칠 수 있기 때문에 이런 상황에서는 for-loop를 사용하도록 하자.

```kt

fun main(args: Array<String>) {
    runBlocking {
        //Producer가 동작하는 CoroutineScope를 지정할 수 있다.
        val producer: ReceiveChannel<Int> = produce<Int> (newSingleThreadContext("new-thread")){
            println(Thread.currentThread())
            for(i in 1..3) send(i)
        }

        producer.consumeEach {
            println(Thread.currentThread())
            println(it)
        }
    }
}

//result
Thread[new-thread,5,main]
Thread[main,5,main]
1
Thread[main,5,main]
2
Thread[main,5,main]
3
```

- 위의 내용을 이용하면 파이프라이닝을 할 수 있다.


```

fun main(args: Array<String>) {
    runBlocking {
        val p1 = produce {
            for(i in 1..100){
                send(i)
                delay(1000)
            }
            close()
        }
        val add2 = pipeline(p1){
            it+2
        }
        val square = pipeline(add2){
            it*it
        }
        square.consumeEach {
            println(it)
        }
    }
}

fun <T,R> CoroutineScope.pipeline(ch: ReceiveChannel<T>, f: (T)->R): ReceiveChannel<R> = produce {
    while(true) {
        send(f(ch.receive()))
    }
}
```

- 기본 channel에는 buffer가 없기 때문에 send를 하더라도 receive해주지 않으면 다음 send를 할 수 없다

```
fun main(args: Array<String>) {
    runBlocking {
        val p1 = produce {
            for(i in 1..100){
                println("Send $i")
                send(i)
            }
            close()
        }
    }
}

//result
Send 1
```

- Channel을 생성할 때 buffer size를 설정할 수 있다. default = 0

```kt
fun main(args: Array<String>) {
    runBlocking {
        val p1 = produce(capacity = 10) {
            for(i in 1..100){
                print("Send $i =>")
                send(i)
                println("Complete!!")
            }
        }
    }
}

//result
Send 1 =>Complete!!
Send 2 =>Complete!!
Send 3 =>Complete!!
Send 4 =>Complete!!
Send 5 =>Complete!!
Send 6 =>Complete!!
Send 7 =>Complete!!
Send 8 =>Complete!!
Send 9 =>Complete!!
Send 10 =>Complete!!
Send 11 =>
```

- **ticker**: 정해진 시간마다 *Unit*을 보내는 channel

```kt
fun main(args: Array<String>) {
    runBlocking {
        val ticker = ticker(1000, 0)
        ticker.consumeEach {
            println(System.currentTimeMillis()%100000)
        }
    }
}

//result
57568
58547
59546
60545
61546
62544
63544
64545
```

Actor
--------

- actor는 코루틴에서 동기화를 위해 많이 사용되는 기능이다. 코루틴에서는 여러 스레드에서 동작하기 때문에 멀티스레드에서 생기는 동시성 문제가 생길 수 있다. 이 때 엑터는 상태 정보에 대한 접근(수정, 읽기)은 단일 스레드로 한정하면서 다른 쓰레드에서 동작하는 코루틴들에게는 이 정보를 채널로 제공하고 수정정보를 받는 기능을 제공한다. 즉, 어떤 코루틴이 동기화 대상을 관리하고 다른 코루틴이 대상의 정보가 필요하거나 상태 변경이 필요한 경우에 관리 코루틴(actor)에게 정보를 전달한다고 보면 된다.
- 아래의 코드는 channel을 통해서 접근되기 때문에 lock없어도 race condition에 의해서 생길 수 있는 문제를 걱정하지 않아도 된다. 특히, context switching없기 때문에 lock을 쓰는 방법보다는 빠르다


```kt
fun main(args: Array<String>) {
    runBlocking {
        val actorCounter = actor<Msg>{
            var cnt = 0
            for (msg in channel){
                when(msg){
                    is UpdateMsg -> cnt += msg.cnt
                    is GetMsg -> msg.responseChannel.complete(cnt)
                }
            }
        }

        launch {
            for(i in 1..3){
                actorCounter.send(UpdateMsg(i))
                val responseChannel = CompletableDeferred<Int>()
                actorCounter.send(GetMsg(responseChannel))
                println("Coroutine1: ${responseChannel.await()}")
            }
        }
        launch {
            for(i in 4..6){
                actorCounter.send(UpdateMsg(i))
                val responseChannel = CompletableDeferred<Int>()
                actorCounter.send(GetMsg(responseChannel))
                println("Coroutine2: ${responseChannel.await()}")
            }
        }

        delay(3000)
        actorCounter.close()
    }
}

sealed class Msg
class UpdateMsg(val cnt: Int): Msg()
class GetMsg(val responseChannel: CompletableDeferred<Int>): Msg()


//result
Coroutine1: 1
Coroutine1: 7
Coroutine2: 7
Coroutine1: 10
Coroutine2: 15
Coroutine2: 21
```

- - -

Flow
=====

- Flow: 비동기로 동작하면서 여러 값을 반환하는 Function
  - flow: Flow를 만드는 Builder
  - asFlow: Collection, Sequence를 Flow로 변환
  - emit: Flow에서 값을 전달하는 함수

```kt
fun main(args: Array<String>) {
    println("Hello World!")

    runBlocking {
        println("start")
        f().collect { println(it) }
        println("end")
    }
    println("Good bye World!")
}


fun f(): Flow<Int> = flow {
    for(i in 1..5) emit(i)
}

// 실행 결과
Hello World!
start
1
2
3
4
5
end
Good bye World!
```

- Flow는 *Cold*이기 때문에 작업(e.g. collect)가 수행되지 않으면 동작하지 않는다.

중간 연산자
--------

- Flow를 다른 Flow로 변환하는 함수.
- suspend가 아니기 때문에 Flow는 즉시 반환하지만 Flow자체는 Cold다.
- 내부에서 suspend function 사용 가능

변환 연산자
--------

- 임의의 값, 횟수 등으로 변환하는 복잡한 역할을 수행
- e.g. transform

```kt

fun main(args: Array<String>) {
    println("Hello World!")

    runBlocking {
        println("start")
        f().transform {
            if(it % 2 == 0) {
                emit(it*2)
                emit(it*3)
            }
        }.collect{ println(it) }
        println("end")
    }

    println("Good bye World!")
}


fun f(): Flow<Int> = flow {
    for(i in 1..5) emit(i)
}

// 실행 결과
Hello World!
start
4
6
8
12
end
Good bye World!
```

- map은 전달 받은 값을 하나의 값으로만 변환해서 전달하지만 transform은 어떤 조건에 맞는 경우에 여러 값을 전달하는 복잡한 연산을 수행할 수 있다  .

종단 연산자
--------

- Flow의 마지막에서 Flow 수집을 시작하도록 하는 연산자.
- e.g. collect, collectLast, toList, first, reduce, fold, collectIndexed

back pressure(배압)
-------------

- Reactor와 다르게 coroutine의 Flow는 back pressure를 제공하지 않는다. 오직 push 방식.
  - back pressure: publisher가 아닌 subscriber가 원하는 경우에 값을 원하는 만큼 값을 전달 받을 수 있는 방법

```kt
fun main(args: Array<String>) {
    runBlocking {
        val startTime = System.currentTimeMillis()
        println("start")
        f().collect{
            delay(3000)
            println("get $it")
        }
        println("end time: ${System.currentTimeMillis() - startTime}")
        delay(100000)
    }
}

fun f(): Flow<Int> = flow {
    for(i in 1..5) {
        delay(1000)
        println("emit $i")
        emit(i)
    }
}

// 실행 결과
start
emit 1
get 1
emit 2
get 2
emit 3
get 3
emit 4
get 4
emit 5
get 5
end time: 20084
```

- 실행 결과를 보면 전체 20초(emit 3초 + get 1초. 5번 반복)이 걸린 것을 볼 수 있다.
- 코루틴에서는 subscriber가 stream을 제어하지 못한다.
