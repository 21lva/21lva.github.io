---
layout: post
title:  "Asyncio"
date:   2019-08-10 02:02:59
author: Inhyuk
category: Python
tags: Python
cover:  "/assets/instacode.png"
name: asyncio.md
---

Asyncio가 필요한 경우
-------------------

- 스레드를 사용하는 선점형 멀티 테스보다 안전함
  - preemptive multitasking은 공유 메모리 접근, race condition등의 문제를 야기할 수 있음
- 동시에 수천 개의 소켓 연결을 처리할 수 있음
- 네트워크 프로그래밍은 계산 위주의 작업보다는 **기다림**의 시간이 길기 때문에 preemptive multitasking이 필요 없다.

- asyncio를 사용한다고 해서 엄청 빨라진다고 할 수는 없다. 운영체제에서 사용하는 스레드의 개수는 제한되어 있기 때문에 병행 소켓 연결을 해주는 asyncio를 사용하는 것.
- asyncio를 사용하더라도 threading은 필요하다. 스레딩의 가장 큰 장점은 일부분의 메모리른 공유하면서 병렬 처리를 한다는 것이다. 이는 오히려 계산 위주의 작업에 매우 큰 도움을 준다
- asyncio는 **GIL(Global Interpretor Lock)** 문제를 제거해주지 못한다. asyncio가 GIL의 영향을 받지 않는 것은 사실이지만, GIL이 멀티쓰레드 프로그램에서만 영향을 미치기 때문이다. GIL의 문제점은 멀티쓰레드에서 멀티 코어를 사용하는 것을 억제하기 때문에 일어나는 것이다. asyncio는 명목상 단일 쓰레드이기 때문에 GIL의 영향을 받지 않는다.
- asyncio는 병렬 프로그래밍을 쉽게 해주는 것이라는 얘기가 있는 이유는 몇 가지 race condition을 제거해주기 때문이다.

스레드
-----

1. 장점
  - 병렬 처리를 통한 성능 향상: 공유 메모리 사용으로 인한 메모리 전송 감소
2. 단점
  - 개발의 어려움
  - 자원 소모
  - 유연하지 않다(동적으로 할당하지 못하기 때문에 멀티프로세스보다도 유연하지 못함)
  - 특정 경우(대규모 병령 처리)에는 오히려 context switching때문에 성능이 저하될 여지가 있음


- 스레드를 쓰고 싶으면 **concurrent.futures**의 **ThreadPoolExecutor**를 사용하도록 하자.

- - -

asyncio
======

- asyncio는 어플리케이션 개발자와 프레임워크 개발자를 대상으로 만들어졌다. 이 부분에서 많은 오해가 발생하는데 asyncio의 공식 문서는 어플리케이션 개발자보다는 프레임워크 개발자에 더 적합하다.

- PEP-492 문서에서는 asyncio에 대한 다음과 같은 내용을 포함하고 있다.
  1. 이벤트 루프 시작하는 방법
  2. async/await
  3. 이벤트 루프에서 사용할 태스크 작성
  4. 태스크의 완료를 기다림
  5. 이벤트 루프 종료


```Python
import asyncio, time


async def async_fun():
    print("Before Async")
    await asyncio.sleep(3)
    print("After Async")


if __name__ == '__main__':
    print("hello")
    asyncio.run(async_fun())
    print("hi")

    #asyncio.run()은 아래의 절차를 축약해서 제공한다
    loop = asyncio.get_event_loop()#event loop 생성 혹은 획득
    task = loop.create_task(async_fun())#태스크 생성 및 등록.
    loop.run_until_complete(task)#태스크 완료 대기. blocking 방식
    pending = asyncio.all_tasks(loop) #모든 태스크 종료 대기.
    for task in pending:#시작되지 않았지만 대기 중인 태스크 취소 요청
        task.cancel()
    #취소 요청 전달한 태스크의 목록 획득
    group = asyncio.gather(*pending, return_exceptions=True)
    #취소 요청 전달된 태스크의 종료를 대기
    loop.run_until_complete(group)
    #이벤트 루프 닫기. asyncio.run은 항상 새로운 이벤트 루프를 열고 닫는다.
    loop.close()

"""
Result:

hello
Before Async
After Async
hi
"""
```

- **run()**은 일련의 과정을 축약해서 제공한다
  - loop = asyncio.get_event_loop() : 코루틴 객체를 실행하기 위한 루프를 얻는다. 같은 스레드에서는 같은 같은 루프 인스턴스를 얻는다(있으면 같은 루프 없으면 생성)
  - task = loop.create_task(async_fun()): 코루틴을 루프에 스케쥴링한다. 반환받은 task 객체로 작업 상태 모니터링
  - loop - run_until_complete(task): 코루틴(task)가 끝날때까지 현재 스레드를 block한다.
    - task 수행은 task 객체의 __step()메서드를 호출하는 것을 의미
    - __step()은 task객체의 coroutine 객체에 send를 호출한다. 이 coroutine 객체는 task의 필드에 등록되어 있다. 이 코루틴은 await 키워드를 마주칠 때까지 수행.
    - 코루틴에서 await를 마주치면 연쇄적으로 *코루틴 체인*을 형성
    - 코루틴 체인은 아무 일 없이 잘 처리되거나 asyncio.sleep과 같은 Sleep 혹은 IO를 await하는 코드를 마주하게 된다. 이 때 future 객체가 생성(혹은 자기 자신)이 전달된다.
    - future 객체는 체인을 따라서 __step()을 호출한 task 객체에 전달되고, task 객체는 이것을 자신의 __fut_waiter라는 필드에 저장한다. 그 후에 task 객체는 future객체의 add_done_callback() 메서드를 호출하여 future 객체가 완료 상태가 될 때 이벤트 루프에게 실행을 예약할 콜백 함수를 등록한다. 이 때 콜백함수로 자기 자신의 __step()을 전달한다.
    - 콜백 등록을 완료한 task 객체는 자신의 실행을 중단하고 이벤트 루프에 제어를 전달.
    - 이벤트 루프는 다시 자신의 큐에서 실행을 예약해둔 task를 골라서 실행시킨다.
    - 이벤트 루프가 자신에게 예약된 테스크가 없으면 select()함수를 이용하여 데이터를 읽거나 쓸 준비가 된 소켓을 찾는다. 준비된 소켓이 있으면 소켓에 바인딩된 future의 결과 값을 업데이트 해준다. 이 때 future에 add_done_callback()의 실행이 이벤트 루프에 예약된다. 콜백 함수의 실행을 예약한다는 의미는 콜백에 등록된 task의 실행을 예약한다는 말이다
    - 이벤트 루프가 task를 실행하게되면 __step()메서드를 호출한다(등록된 콜백 자체가 __step()을 호출하도록 되어 있다). __step()은 먼저 __fut_waiter에 바인딩된 future와 연결을 해제한다. 그 후에 send()메서드를 호출해서 자신의 실행을 재개한다. 이때 바인딩이 해제된 future의 자기 자신을 yield하는 __await__()까지 가게 된다. IO의 경우에는 소켓에서 읽은 데이터를 return하고 sleep은 바로 return
    - 이벤틀 루프가 위의 작업을 반복하다보면 결국에 task가 등록된 최초의 코루틴을 return하는 경우가 발생한다. task의 __step()은 StopIteration 예외를 발생시킬 것이고 그 예외 객체의 value 필드 값으로 자기 자신(task)의 결과를 업데이트 한 후 자신의 실행을 종료한다. loop.run_until_complete() 함수의 실행이 끝나는 시점이다. 이때 등록된 task의 결과를 전달한다
  - pending = asyncio.all_tasks(loop): loop.stop()이나 process signal에 의해 block 상태가 풀린 이후에 pending 상태(완료되지 않은)인 작업들을 수합한다,
  - for문: pending 상태인 작업을 모두 종료 시킨다.
  - group = asyncio.gather(*pending): 취소 이후로도 진행중인 작업을 모은다.
  - loop.run_until_complete(group): 아직 진행중인 작업이 모두 끝날때까지 기다린다.
  - loop.close(): 루프의 모든 대기열을 비우고 executor을 종료. 이제 해당 루프는 재사용 불가능하다.

- 이벤트 루프는 무한히 작업을 하나씩 수행하는 로직이다. 각 스레드마다 하나씩 생성
- nodejs는 내부의 이벤트 루프 동작 방식을 몰라도 잘 사용할 수 있지만, asyncio는 이벤트루프 및 태스크의 lifecycle에 대해 잘 알아야한다.

- I/O bound 함수들은 await을 이용하여 적절히 context switching을 제어해야 한다. 파이썬에서는 이 작업을 자동으로 해주지 않기 때문에 직접 작성해야한다. async def를 사용하지 않으면 concurrent.futures의 ThreadPoolExecutor(멀티 스레드 방식)과 ProcessPoolExecutor(멀티 프로세스 방식)등을 이용하여 처리한다.
  - loop.run_in_executor(None, blocking_func): blocking_func이 main 쓰레드를 block하는 경우에 이 함수를 기본 익스큐터에서 실행되도록 하여 메인 쓰레드를 block하지 않도록 한다. run_until_complete()을 통해서 이벤트 루프에 전달한다. 반환값은 **Future**로 코루틴 함수에서 호출한 경우에는 *await*를 통해서 blocking_func의 실행을 기다릴 수 있다. executor에 할당된 작업은 *run_until_complete*를 실행해야 시작된다.

```py
import asyncio
import time


async def async_func():
    print("Before")
    await asyncio.sleep(1)
    print("After")


async def blocking_func():
    time.sleep(30)
    print("FINISH SLEEP")


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    t1 = loop.create_task(blocking_func())
    t2 = loop.create_task(async_func())
    print("CREATE!!!")
    group = asyncio.all_tasks(loop)
    loop.run_until_complete(asyncio.gather(*group))
```

- 위의 코드에서 예상한 동작은 task1과 task2가 비동기로 동작하는 것이다. 그러나 time은 쓰레드를 block하므로 loop가 쓰레드가 block이 되고 결국에는 해당 루프에서 동작하는 모든 코루틴이 동작을 멈추게 된다. 이럴 때는 다른 executor에서 동작하게 해야한다.

```py
import asyncio
import time

async def async_func():
    print("Before")
    await asyncio.sleep(1)
    print("After")


def blocking_func():
    time.sleep(10)
    print("FINISH SLEEP")


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    t1 = loop.run_in_executor(None, blocking_func)#executor에 None을 전달하면 기본 executor가 사용된다.
    t2 = loop.create_task(async_func())
    print("CREATE")
    group = asyncio.all_tasks(loop)
    loop.run_until_complete(asyncio.gather(*group))
    print("END WAIT")

""" result
CREATE
Before
After
END WAIT
FINISH SLEEP
"""
```

asyncio의 계층적 구조
----------------

- asyncio를 사용하려면 어느 부분부터 사용해야 하는지에 대해 명확하지 않을때가 있다.
- 아래는 asyncio의 계층적 구조이고, 특정 부분부터 직접 작성하면 된다

  1층(코루틴): async def, async with, await 등. 서드파티 프레임 설계의 시발점. 가장 기초 단계
  2층(이벤트 루프): 코루틴은 자신을 실행 시킬 루프 없이는 의미 없음. 2층에서는 이러한 루프(BaseEventLoop) 혹은 interface(AbstractEventLoop)를 제공. uvloop 라이브러리는 asyncio의 표준 라이브러리가 제공하는 이벤트 루프보다 빠른 성능을 보인다. 특히, asyncio의 2층에 대해서만 플러그인 되어 있기 때문에 교체하기 쉽다.
  3층(Futre),4층(Task): Future는 이벤트 루프에서 실행 중인 작업(Task가 아님)로 알림을 통해 결과를 반환. Task는 이벤트에서 실행중인 **코루틴**. 어플리케이션 개발자는 Task를 더 많이 쓰고 반대로 프레임워크 개발자는 Future를 더 이용한다.
  5층(쓰레드, 프로세스)
  6층(tool): asyncio.Qeueu와 같은 추가적인 비동기 기반 도구를 제공. 코루틴에 데이터를 전달할때 사용. get(main 스레드를 blocking하기 때문에 코루틴 내에서 직접 사용하는 것은 지양)과 put을 이용한다.
  7층(네트워크: 트랜스포트),8층(TCP, UDP),9층(Stream): Streams API를 제공하는 계층. 소켓 통신을 처리하는 간단한 기능 제공. asyncio와 호환되는 소켓 통신용 라이브러리(aiohttp 등)를 이용하면 이 계층들을 직접 사용하지 않아도 된다.

- - -

코루틴
=====

```Python
async def f():
    return 23131
```

- f는 코루틴 함수. f()를 통해서 얻는 객체가 코루틴이다(generator도 함수가 제너레이터인 것이 아니라 함수를 호출해서 생성된 객체가 제너레이터)
- 코루틴: 완료되지 않은채 일시 정지된 함수를 재개할 수 있는 기능을 가진 객체. 코루틴이 반환할때(될때가 아니다)는 **StopIteration** 예외가 발생한다. 이벤트 루프는 이 예외를 받아서 코루틴을 전환한다.
- send()와 StopIteraion이 코루틴의 시작과 끝을 의미한다. 이 부분은 이벤트 루프가 다룬다.
- python3.5부터 coroutine을 명시적으로 지정하는 async, await 키워드가 도입됨.
- 기존의 yield를 사용하는 coroutine을 generator based coroutine이라 하고 async, await을 사용하는 coroutine을 native coroutine이라고 함. 전자의 경우에는 함수 내에 yield from이 있을 때 코루틴 함수가 되지만 후자의 경우에는 async 키워드가 달려있으면 함수 내에 await가 있는 것과 상관 없이 코루틴 함수가 된다. async로 만든 코루틴 함수에서는 yield from을 사용할 수 없다.


```py
async def async_func():
    print("Before")
    print("After")
    return "abc"


if __name__ == '__main__':
    x = async_func()
    print(type(x)) #x는 코루틴이다
    try:
        x.send(None) #x를 초기화한다. 이 작업은 eventloop에서 이루어진다.
    except StopIteration as e: #코루틴이 반환할 때 StopIteraion 발생.
        print(e.value)#코루틴의 반환값은 e.value에 저장된다.

"""result
<class 'coroutine'>
Before
After
abc
"""
```

- await: awaitable(task, future, coroutine)을 받아서 루틴을 시작해서 결과를 반환하게 한다.
- await는 yield from의 대체가 될 수 있지만 yield의 대체는 될 수 없다. async-await는 제너레이터와 코루틴을 분리하면서 오직 비동기 프로그래밍을 위한 용도로 설계되었다.

- async-await가 추가되면서 data model이 확장되었다
  1. awaitable object
    - __await__()가 구현된 객체. ex) async def 함수를 호출하면 native coroutine
    - object.__await__(self): iterator를 반환. await에서 사용된다. Future의 경우도 __await__()가 구현되어 있어서 await에서 사용가능
  2. coroutine object: awaitable 객체. send, throw, close가 구현되어 있다
  3. asynchronous iterators: 기존의 iterator의 메서드와 대응되는 __aiter__(), __anext__()가 구현된 객체. async for 에서 사용 가능
  4. asynchronous context managers: 기존의 with에서 사용하는 context manager와 비슷하게 __aenter__(), __aexit__()가 구현된 객체. async with에서 사용


- 기본적인 eventloop: await를 만나면 해당 awaitable은 이벤트 루프에 await뒤에 나오는 awaitable을 넘겨준다. 이벤트 루프는 전달받은 awaitable을 (코루틴인 경우에 태스크로 변경하고) 스케쥴링한다.
- coroutine.cancel(): 이벤트 루프에서는 내부적으로 coroutine.throw()을 통해서 코루틴 내부에서 CancelledError를 발생시킨다.

```py
async def async_func():
    try:
        print("Before")
        while True:
            await asyncio.sleep(0)
    except asyncio.CancelledError:
        print("Cancelled")


if __name__ == '__main__':
    x = async_func()
    print(type(x))
    x.send(None)
    x.throw(asyncio.CancelledError) # 코루틴내에서 CancelledError를 다룬후에 정상 종료(StopIteraion)된다.

"""result
<class 'coroutine'>
Before
Cancelled
Traceback (most recent call last):
  File "/pythonProject/main.py", line 17, in <module>
    x.throw(asyncio.CancelledError)
StopIteration
"""
```

- - -

이벤트 루프
========

- 이벤트 루프는 코루틴간의 전환, StopIteraion 처리, 소켓과 파일 디스크립터의 이벤트 수신 등을 처리

코루틴안에서 asyncio.get_running_loop vs 어디서나 asyncio.get_event_loop
----------------

- get_event_loop: **동일한 스레드** 내에서만 동작한다. new_event_loop를 호출하여 새로운 루프를 생성하고, set_event_loop를 통해서 해당 루프를 스레드의 새로운 루프 인스턴스로 지정해야 한다.
- get_running_loop: 코루틴이나 테스크에서 호출 가능. 현재 동작중인 이벤트 루프를 얻는다. 현재 동작중인 loop을 관리할 필요 없이 편하게 create_task를 할 수 있게 해준다.

```Python
async def inner_async_f(i):
    print(f"Inner: {i}")

async def out_async_f():
    print("Before Async")
    in_loop = asyncio.get_running_loop()
    for i in range(2):
        in_loop.create_task(inner_async_f(i))
    print("After Async")


async def out_async_f2():
    #위의 코루틴 함수과 같다
    print("Before Async")
    for i in range(2):
        #위 코루틴의 loop를 얻고, task를 전달하는 기능을 제공한다
        asyncio.create_task(inner_async_f(i))
    print("After Async")

if __name__ == '__main__':
    print("hello")
    asyncio.run(out_async_f())
    asyncio.run(out_async_f2())
    print("hi")

""" result
hello
Before Async
After Async
Inner: 0
Inner: 1
Before Async
After Async
Inner: 0
Inner: 1
hi
"""
```

- 아래의 내용은 *create_task*함수의 구현이다. 내부에서 get_running_loop을 얻어서 사용한다.

```py
def create_task(coro, *, name=None):
    """Schedule the execution of a coroutine object in a spawn task.

    Return a Task object.
    """
    loop = events.get_running_loop()
    task = loop.create_task(coro)
    _set_task_name(task, name)
    return task
```


- - -

Task & Future
==========

[concurrent.future](https://soooprmx.com/concurrent-futures/)

- Task는 Future의 하위 클래스. Future가 제공하는 기능(완료 상태를 나타내고 루프에 의해 관리됨)과 함께 동작을 제공한다. 대부분의 경우에는 Task 사용. 익스큐털르 사용할 때에는 Task가 아닌 Future를 반환 받는다.
- Future는 작업이 끝났는지 아닌지를 확인할 수 있다.
  - done: 끝났는지 확인
  - cancel: 작업 취소
  - set_result: 작업 끝나고 전달할 result를 변경
  - result(): 작업 후 result 값 반환

- loop.create_task vs asyncio.ensure_future:
  - asyncio.ensure_future(coro_or_future): 코루틴을 전달하면 Task 생성, Future을 전달하면 Task로 변경 없이 동일한 인스턴스 반환. 이렇게 하나의 함수가 두 가지 타입을 전달하는 것은 프레임워크 설계자를 위해 만들어진 **helper function**이기 때문이다. ex) asyncio.gather는 awaitable 객체를 전달 받고 내부적으로 ensure_future를 통해서 변환한다.
  - 3.7 이상에서는 asyncio.create_task()가 asyncio.ensure_future를 대신한다
  - 그러므로 일반 개발자는 create_task를 사용하자.


- asyncio.Future vs concurrent.futures.Future: 전자는 await 지점에 멈춰있는 상태이고, 후자는 지금 동작하고 있는 상태이다.

- - -

async 기능들
============


async with
----------

- 비동기 context 관리자: 특정 객체의 생명 주기를 적절한 범위 내에서 관리하고자 할 때 사용
- context 관리자의 동작은 메서드 호출로 이루어짐.
- *__enter__*와 *__exit__*내에서 await로 비동기 처리를 해야하는 경웅에만 사용하면 된다.

```py
class AsyncCon:
    def __init__(self, host, port):
        self._host = host
        self._port = port

    async def __aenter__(self):
        self.conn =  await self.get_conn()
        print("Connection")

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Finish Connection")

    async def get_conn(self):
        print(f"Connecting to {self._host}:{self._port}")
        await asyncio.sleep(3)
        return "conn"


async def async_f():
    print("Before connection")
    async with AsyncCon("http://localhost", 12112):
        print("Do job")


if __name__ == '__main__':
    print("hello")
    asyncio.run(async_f())
    print("hi")

"""result
hello
Before connection
Connecting to http://localhost:12112
Connection
Do job
Finish Connection
hi
"""
```

- 이 방식말고도 **contextmanager decorator**를 사용하는 방법도 존재한다.

```py
@contextlib.asynccontextmanager
async def async_context(host, port):
    print(f"Connecting to {host}:{port}")
    print("Connection")
    await asyncio.sleep(3)#임시
    yield "conn"
    print("Finish Connection")


async def async_f():
    print("Before connection")
    async with async_context("http://localhost", 12112):
        print("Do job")


if __name__ == '__main__':
    print("hello")
    asyncio.run(async_f())
    print("hi")

"""result
hello
Before connection
Connecting to http://localhost:12112
Connection
Do job
Finish Connection
hi
"""
```

- 아래의 방식은 blocking호출을 하는 코드를 다른 executor에서 동작시키는 방식이다.

```py
def blocking_fun():
    time.sleep(3)
    return "conn"

@contextlib.asynccontextmanager
async def async_context(host, port):
    print(f"Connecting to {host}:{port}")
    print("Connection")
    loop = asyncio.get_event_loop()
    yield await loop.run_in_executor(None, blocking_fun)
    print("Finish Connection")


async def async_f():
    print("Before connection")
    async with async_context("http://localhost", 12112):
        print("Do job")


if __name__ == '__main__':
    print("hello")
    asyncio.run(async_f())
    print("hi")

"""result
"""
```

async for
----------

- 비동기 이터레이터(__next__ 매직 메서드가 코루틴 함수인 경우)에 대해 **async for** 를 사용하려 하면 다음과 같은 명세를 지켜야한다(PEP 492)
  - **def __aiter__()**를 구현해야한다
  - **__aiter__()**는 **async def __anext__()**를 구현한 객체를 반환해야한다
  - **__anext__**는 반복의 각 단계에 대한 값을 반환하고, 반복이 끝나면 StopAsyncIteration을 발생시켜야 한다.

```Python
import asyncio, time
import random

class AsyncCls:
    def __init__(self, range=3):
        self.range = range

    def __aiter__(self):
        self.r = iter(range(0, self.range))
        return self

    async def __anext__(self):
        try:
            x = next(self.r)
            await asyncio.sleep(random.randint(0, 3))
            return x
        except StopIteration:
            raise StopAsyncIteration


async def x():
    async for value in AsyncCls():
        print(value)
        print("GET")


if __name__ == '__main__':
    print("hello")
    asyncio.run(x())
    print("hi")

"""
Result:

hello
0
GET
1
GET
2
GET
hi
"""
```

- 비동기 처리를 순서대로 처리한다


비동기 제너레이터
---------

- 비동기 제너레이터: **async def**안에 **yield**가 포함된 구조


```Python
import asyncio, time
import random


async def gen():
    for i in range(4):
        x = random.randint(0,3)
        print(f"SLEEP for {x}: {i}")
        await asyncio.sleep(x)
        print("END")
        yield i


async def x():
    async for value in gen():
        print(value)
        print("GET")


if __name__ == '__main__':
    print("hello")
    asyncio.run(x())
    print("hi")

"""
Result:

hello
SLEEP for 1: 0
END
0
GET
SLEEP for 2: 1
END
1
GET
SLEEP for 0: 2
END
2
GET
SLEEP for 2: 3
END
3
GET
hi
"""
```

async comprehension
------------------

- 비동기 함수, 제너레이터 등을 list, set comprehension을 적용하는 것

```Python
import asyncio, time
import random


async def f(i):
    await asyncio.sleep(random.randint(0,3))
    return i


async def gen():
    for i in range(4):
        await asyncio.sleep(random.randint(0, 3))
        yield f, i


async def main():
    print([await f(x) async for f,x in gen()])

if __name__ == '__main__':
    print("hello")
    asyncio.run(main())
    print("hi")

"""
Result:

hello
[0, 1, 2, 3]
hi
"""
```

- - -

시작, 종료
=======

- asyncio 어플리케이션을 시작하는 가장 일반적인 방법은 **main() 코루틴 함수**를 정의하고 그 것을 asyncio.run()에서 실행하는 것이다.

```Python
async def main():
    pass

if __name__=="__main__":
    asyncio.run(main())
```

- run의 종료는 위에서 설명했듯이
  - 모든 pending 작업에 취소 시그널을 보내서 CancelledError를 발생시킨다.
  - 모든 task를 group task로 수집한다
  - 해당 task들을 run_until_complete로 끝날때까지 기다린다. 이는 task들이 스스로 CancelledError를 처리하는 것을 기다리는 것이다.

- 위의 방식은 자칫 주의를 기울이지 않으면 **The event loop is still running** 이라는 문구를 보게 될 것이다. 이는 eventloop에 아직 태스크가 남아있는데 종료를 시도했기 때문이다.

```Python
#위의 문구가 나오는 코드

async def f(x):
    await asyncio.sleep(x)


async def main():
    loop = asyncio.get_event_loop()
    t1 = loop.create_task(f(1))
    t1 = loop.create_task(f(3))
    loop.run_until_complete(t1)
    loop.close()


if __name__ == '__main__':
    print("hello")
    asyncio.run(main())
    print("hi")
```

- 종료를 제대로 하고 싶으면 run에서 스스로 하도록 위임하자. loop.run_until_complete를 두 번 부르는 것은 오류를 발생시킨다. ex) asyncio.run내에서 loop.run_until_complete를 시행.
- 아래는 제대로 종료된다.

```py
async def f(x, name):
    print(f"[{name}] Before sleep {x} seconds")
    for i in range(x):
        print(f"[{name}] await {i} seconds")
        await asyncio.sleep(1)
    print(f"[{name}] After sleep {x} seconds")


async def main():
    task1 = asyncio.create_task(f(3, "first"))
    task2 = asyncio.create_task(f(20, "second"))#시행은 되지만 await되지 않는다.
    loop = asyncio.get_running_loop()
    await asyncio.gather(task1, loop=loop)
    print("done")


if __name__ == '__main__':
    print("hello")
    asyncio.run(main())
    print("hi")

"""result
hello
[first] Before sleep 3 seconds
[first] await 0 seconds
[second] Before sleep 20 seconds
[second] await 0 seconds
[first] await 1 seconds
[second] await 1 seconds
[first] await 2 seconds
[second] await 2 seconds
[first] After sleep 3 seconds
[second] await 3 seconds
done
hi
"""
```

gather(..., return_exceptions=True)
----------------------

- run_until_complete는 Future에 대해서 동작. gather는 Future를 반환.
- run_until_complete에서 Future를 수행하던 중에 예외가 발생되면 run_until_complete는 그 예외를 전달하고 이벤트 루프는 중지됨.
- 만약에 Future가 group Future이고 해당 하위 Future중 하나에서 exception이 발생하고 그것을 제대로 핸들링 못하면 나머지 Future들이 제대로 종료되기 전에 이벤트 루프가 중지된다. 이 때 예외는 CancelledError도 포함이다
- 아래의 코드에서는 f1 task에 의해 f2 task가 제대로 동작되지 않는다.

```py
async def f1():
    for i in range(3):
        print(f"First: {i}")
        await asyncio.sleep(1)
    raise ValueError


async def f2():
    try:
        for i in range(10):
            print(f"Second: {i}")
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print("Second is cancelled")


async def main():
    task1 = asyncio.create_task(f1())
    task2 = asyncio.create_task(f2())
    await asyncio.gather(task1, task2)


if __name__ == '__main__':
    print("hello")
    asyncio.run(main())

"""result
hello
First: 0
Second: 0
First: 1
Second: 1
First: 2
Second: 2
ValueError
Second: 3
Second is cancelled
Traceback (most recent call last):
    <------생략------>
    raise ValueError
"""
```

- 하나의 Future에 의해 다른 Future들이 영향을 받으면 안되는 경우가 있다. 이럴 때 *return_exceptions=True*(default = False)를 이용한다. 해당 flag는 Future 그룹의 하위 Future에서 발생하는 예외를 return 값으로 처리하도록 설정한다.

```py
async def f1():
    for i in range(3):
        print(f"First: {i}")
        await asyncio.sleep(1)
    raise ValueError


async def f2():
    try:
        for i in range(5):
            print(f"Second: {i}")
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print("Second is cancelled")


async def main():
    task1 = asyncio.create_task(f1())
    task2 = asyncio.create_task(f2())
    x = await asyncio.gather(task1, task2, return_exceptions=True)
    print(x)


if __name__ == '__main__':
    print("hello")
    asyncio.run(main())

"""result
hello
First: 0
Second: 0
First: 1
Second: 1
First: 2 ==> 여기서 에러 발생하지만 다른 Future에는 영향을 미치지 않는다.
Second: 2
Second: 3
Second: 4
[ValueError(), None]
"""
```

Signal
------

- **KeyboardInterrupt**(ctrl+c)를 통해서 이벤트 루프를 종료하면 다음과 같이 asyncio.run이 내부로 **CancelledError**를 전달한다.

```py
async def f(x, name):
    try:
        print(f"[{name}] Before sleep {x} seconds")
        for i in range(x):
            print(f"[{name}] await {i} seconds")
            await asyncio.sleep(1)
        print(f"[{name}] After sleep {x} seconds")
    except asyncio.CancelledError:
        print("Cancel inner task")


async def main():
    try:
        task1 = asyncio.create_task(f(30, "first"))
        loop = asyncio.get_running_loop()
        await asyncio.gather(task1, loop=loop)
        print("done")
    except asyncio.CancelledError:
        print("Cancel main task")


if __name__ == '__main__':
    print("hello")
    try:
        asyncio.run(main())
        print("Finished")
    except KeyboardInterrupt:
        print("Interrupted")

"""result
hello
[first] Before sleep 30 seconds
[first] await 0 seconds
[first] await 1 seconds
Cancel inner task
Cancel main task
Interrupted
"""
```

종료 중 executor 기다리기
--------------

- run_in_executor 시작시킨 작업의 경우에는 await를 하지 않으면 문제가 생긴다. 이 문제는 run_in_executor가 Task가 아닌 Future를 반환하기 때문이다. 진행 중인 task에 포함되지 않으므로 asyncio.run의 run_until_complete에서 대기하지 않는다.
- 파이썬 3.8이하의 경우에는 loop.close()가 익스큐터 작업이 종료되기 까지 기다리지 않기 때문에 run_in_executor에서 반환한 Future가 오류를 발생시킴. 3.9부터는 잘된다
- 아래의 코드를 3.8에서 동작시키면 에러가 발생한다.

```py
def blocking_fun():
    print("start sleep")
    time.sleep(5)
    print("finish sleep")


async def async_fun():
    print("f2 - start")
    await asyncio.sleep(2)
    print("f2 - end")


async def main():
    lp = asyncio.get_running_loop()
    t1 = lp.create_task(async_fun())
    t2 = lp.run_in_executor(None, blocking_fun)
    await t1
    #await t2 => 이것을 추가하면 에러가 발생하지 않는다.


if __name__ == '__main__':
    asyncio.run(main())

"""Result
start sleep
f2 - start
f2 - end
finish sleep
exception calling callback for <Future at 0x105698af0 state=finished returned NoneType>
<-----생략---->
RuntimeError: Event loop is closed
"""
```
