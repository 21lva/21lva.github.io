---
layout: post
title:  "Generator"
date:   2022-06-17 02:02:59
author: Inhyuk
category: Python
tags: Python
cover:  "/assets/instacode.png"
name: generator.md
---

iterable & iterator
=====

iterable
-----

- 반복가능한 객체로 *__iter__*를 구현하여 for, while문에서 사용 가능
- *__iter__*를 통해서 iterator를 얻을 수 있음.

```py
sample_list = [1,2,3,4]
# sample_list.__next__() => 오류 발생
for i in sample_list: #이터러블의 __iter__를 호출해서 iterator를 얻는다.
    print(i)

for i in range(3): # range는 iterable. 값을 필요시에 하나씩 생성
    print(i)
```

- 시퀀스 객체: iterable 중 indexing과 slicing이 지원되는 객체. ex) list, tuple, range, str. __getitem__()와 __len__()을 지원
- for문에서 *__iter__()*를 통해서 iterator를 사용한다. 그러나 해당 매직 메서드가 없지만 sequence 객체인 경우에는 *IndexError*가 발생할 때 까지 순서대로 값을 제공한다.


iterator
--------

- 값을 차례대로 획득할 수 있는 객체
- *__next()__*를 구현한 객체

```py
class SampleIterator:
    def __init__(self):
        self._idx = 0

    def __iter__(self):#보통은 iterator가 iterable이기도 하다.
        return self

    def __next__(self):
        if self._idx >= 10:
            raise StopIteration

        self._idx+=1
        return self._idx


if __name__ == '__main__':
    for value in SampleIterator(): iterator를
        print(value)

"""result
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
"""
```


- - -


제너레이터
=========

- 고성능 + 적은 메모리 사용을 이용한 이터러블을 생성해주는 객체


```py
def generator():
    yield "1"
    yield "2"
    yield "3"
    yield "4"


if __name__ == '__main__':
    for i in generator():
        print(i) # 1,2,3,4
    x = generator()
    print(x.__next__()) # 1
    print(x.__next__()) # 2
    print(x.__next__()) # 3
    print(x.__next__()) # 4
    print(x.__next__()) # StopIteration 오류 발생!!
```


- - -

coroutine
-----------

- coroutine: sub routine이 메인 루틴과 대등한 관계에서 동작하는 루틴. 서브루틴이 종료되지 않은 상태에서 메인 루틴의 코드를 실행하다가 다시 돌아와서 서브루틴의 코드를 수행. 즉, 메인 루틴이 서브루틴을 여러번 접근할 수 있음(entry point가 여러 곳인 함수)
- 코루틴은 논블로킹이다.([예제 코드](https://github.com/werellel/blog))
  - 블로킹: 호출된 함수가 자신이 한 일을 *전부* 끝낼 때까지 제어권을 갖고 있고 호출한 함수는 기다린다.
  - 논블로킹: 호출된 함수가 자신이 한 일을 *전부* 끝내지 않아도 제어권을 호출한 함수에게 전달한다.
  - 동기: 시간관점. 작업의 시작과 끝을 맞춘다. 호출한 함수가 호출된 함수의 종료를 신경쓴다
  - 비동기: 동기가 아닌 작업

[비동기, 블로킹](https://exmemory.tistory.com/78)

```
<기존>
메인 루틴        서브루틴
|
| ----------->  |
                |
대기             |
                |
| <---------- (종료)
V

<코루틴>
메인 루틴        서브루틴
| --최초 호출---> |
대기             |
| <----------  중지
|              대기
호출 ----------> |
대기             |
| <----------  중지
|              대기
호출 ----------> |
대기             |
| <---------- (종료)
V
```
- PEP-342부터 코루틴을 위한 3가지 메서드를 제공
  1. close()
  2. throw(ex_type[, ex_value[, ex_traceback]]):
  3. send(value)

#### close

- 제너레이터에서 *GeneratorExit* 예외를 발생시키고 이 예외를 처리하지 않으면 제너레이터가 값을 더 생성하지 않고 반복이 중지됨.

```py

def generator():
    try:
        yield "1"
        yield "2"
        yield "3"
        yield "4"
    except GeneratorExit as e:
        print("Exit")


if __name__ == '__main__':
    x = generator()
    print(x.__next__()) # 1
    print(x.__next__()) # 2
    x.close()           # Exit
```

#### throw(ex_type[, ex_value[, ex_traceback]])

- 현재 제너레이터가 중단된 위치에서 주어진 예외를 던진다.
- 예외를 except로 처리하지 않으면 그대로 예외가 호출자에게 전달된다.

```py
def generator():
    while True:
        try:
            print("before")
            x = yield "1"
            print(x)
        except ValueError as e:
            print("Exit")


if __name__ == '__main__':
    x = generator()
    print(next(x))
    x.send(12)
    x.send(1)
    x.send(3)
    x.send(4)

"""result
before
1
12
before
1
before
3
before
4
before
"""
```

#### send(value)

- 제너레이터에서 *value*를 인자값으로 전달할 수 있게 해준다. (next는 불가능)
- 보통은 *place_holder = yield return_value* 형식으로 사용되는데, return_value를 *next*를 호출한 쪽에 전달하고 기다린다. 호출자는 *send(value)*를 통해서 제너레이터를 다시 동작시킨다. 이때 value는 제너레이터 내의 place_holder로 전달된다. send를 수행하면 다음 yield 이전까지 수행한다.
- 제너레이터에 send를 통해서 값을 전달하려면 무조건 next를 먼저 수행해야 한다.


```py

def generator():
    while True:
        try:
            x = yield "1"
            print(x)
        except ValueError as e:
            print("Exit")


if __name__ == '__main__':
    x = generator()
    print(next(x)) # 1
    x.send(12) # 12
```

yield from
----------

- generator내에서 generator를 사용할 수 있는 방법

```py
def generator():
    x = [1,2,3]
    for i in x:
        yield i


def generator2():
    x = [1,2,3]
    yield from x


def inner_generator():
    for i in range(3):
        yield i


def generator3():
    yield from inner_generator()


if __name__ == '__main__':
    for i in generator3():
        print(i)

    for i in generator():
        print(i)

    for i in generator2():
        print(i)

"""result
0
1
2
1
2
3
1
2
3
"""
```

- 위의 내용은 가장 간단한 *yield from* 사용 방법이지만, 아래의 코드는 *yield from*을 응용한 방법이다.

```py
def sequence(start, end):#시퀀스 제너레이터는 range iterable에 위임한다
    print(f"{start} -> {end}")
    yield from range(start, end)
    return end


def main():#메인은 시퀀스 제너레이터에게 위임한다
    step1 = yield from sequence(0, 3)
    step2 = yield from sequence(step1, 5)
    return step1+step2 # yield from을 이용하면 코루틴 종료시 최종 값을 반환 가능


if __name__ == '__main__':
    for i in main():
        print(i)

"""result
0 -> 3
0
1
2
3 -> 5
3
4
"""
```

- *yield from*에서 자동으로 예외를 처리하지 않으면 직접 호출 제너레이터가 처리해야한다.
- 코루틴은 제너레이터지만 반복을 위해서 만들어진 기존의 제너레이터와 달리 나중을 위해서 코드의 실행을 멈추는 데에 집중한다.

```py
def sequence(start, end, name):
    print(f"{start} -> {end}")
    for i in range(start,end):
        try:
            x = yield i
            print(f"get: {x}")
        except ValueError:
            print(f"{name} => ERROR!!!!!")


def main():
    yield from sequence(0, 3, "first")
    yield from sequence(0, 3, "second")


if __name__ == '__main__':
    x = main()
    print(next(x))
    print(next(x))
    print(x.send("a"))
    x.throw(ValueError)
    x.throw(ValueError)
"""result
0 -> 3
0
get: None
1
get: a
2
first => ERROR!!!!!
0 -> 3
second => ERROR!!!!!"""
```

- 위의 예제에서 몇 가지 사실을 알 수 있다.
  1. send와 throw를 외부 제너레이터에 대하여 호출하였지만, 내부 제너레이터에 전달된다.
  2. 내부 제너레이터는 순서대로 호출되며 send, throw도 현재 사용중인 내부 제너레이터에 전달된다.

async, await
--------

- await: *yield from*을 대체하기 위한 용도. 뒤에는 awaitable 객체가 전달된다(코루틴은 awaitable).
- async: 코루틴을 정의하는 새로운 방법. 기존의 *@coroutine*을 대체. 호출 시에 코루틴 객체를 전달한다.
- 논블로킹을 넘어서 비동기 프로그래밍을 하고 싶으면 **asyncio**와 같은 이벤트 루프 라이브러리를 이용해야한다. 코루틴은 이벤트 루프에 의해서 호출되며 진행중에 *await <다른 코루틴>*을 통해서 코루틴은 제어권을 이벤트 루프에 다시 전달한다.

```
<asyncio>
이벤트루프       코루틴
| --최초 호출---> |
대기             |
| <----------  await
|              대기
호출 ----------> |
대기             |
| <---------- await
|              대기
호출 ----------> |
대기             |
| <---------- return
V
```
