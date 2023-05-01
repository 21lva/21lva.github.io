---
layout: post
title:  "Decorator "
date:   2019-08-10 02:02:59
author: Inhyuk
category: Python
tags: Python
cover:  "/assets/instacode.png"
name: decorator.md
---

Decorator
=========


- 데코레이터는 데로케리어 이후에 나오는 것을 데코레이터의 첫 parameter로 하고 결과값을 반환한다
- 데코레이터는 **Closure**를 이용한다
- 데코레이터의 활용 가능성
  - 파라미터 검사 및 변환
  - 재시도 로직 구현
  - 캐싱 작업
  - 반복 작업 단순화
  - tracing(추적): 함수의 지표를 모니터링

- 잘못 설계된 데코레이터는 코드의 복잡성을 높이고, 가독성은 낮춘다.
- 데코레이터는 사용하는 클래스에서 데코레이터의 구현을 알지 못해도 동작하도록 해야한다(캡슐화)
- 데코레이터는 다른 객체나 클래스와 독립적이어야 한다
- 데코레이터는 재사용성이 높은 경우에 사용되어야 한다

- - -

Closure
==========

- [참고 자료](http://schoolofweb.net/blog/posts/%ED%8C%8C%EC%9D%B4%EC%8D%AC-%ED%81%B4%EB%A1%9C%EC%A0%80-closure/)
- 클로져는 함수를 상태와 같이 저장한다.
- first class function을 지원하는 언어의 네임 바인딩 기술


```Python
def closure_fuction(x):
    val1 = x
    def inner():
        print(val1)
    return inner

x = closure_fuction(12) # result: 12
y = closure_fuction("hi") # result: hi
x() # result: 12
y() # result: hi
```

- - -

데코레이터 종류
=======

함수 데코레이터
---------

- 모든 종류의 로직 적용 가능
- 패러미터 유효성 검사, 기능 정의, 함수 결과 캐싱 등을 수행 가능

```Python
# 기본적인 함수 데코레이터
@dec1
def original():
  ...

# 위는 아래와 같은 역할을 한다
original = dec1(original)

# 패러미터 유효성 검사를 진행하는 데코레이터
def check_parameter(operation):
    @wraps(operation)
    def wrapped(*args, **kwargs):
        # 검사 진행
        return operation(*args, **kwargs)

@check_parameter
def fun1(x,y):
    ...
```

클래스 데코레이터
-----------

- 클래스에도 데코레이터 적용 가능
- 데코레이터 함수의 파라미터로 함수가 아닌 클래스를 받는다
- 많은 개발자들은 클래스 데코레이터가 클래스 내부의 속성과 메서드를 변경하기 때문에 사용에 주의해야 한다고 말한다

- 장점
  - 클래스 데코레이터는 코드 재사용성을 높여준다(ex. 여러 클래스에 적용할 검사하는 코드의 중복성 제거)
  - 클래스 데코레이터를 사용하면 여러 클래스가 특정 인터페이스 기준을 따르게 할 수 있다.
  - 작은 클래스를 생성 후에 추가적인 기능을 데코레이터를 이용하여 보강 가능

```Python
# Login시에 직렬화가 필요한 경우
class LoginSerializer:
    def __init__(self, event):
        self.event = event

    def serialize(selft):
        # 생략

class LoginEvent:
    SERIALIZER = LoginSerializer

    def serialize(self):
        return self.SERIALIZER(self).serialize()
```

- 위 코드의 문제점은 다음과 같다
  - 이벤트 클래스와 직렬화 클래스가 1:1 대응이므로 직렬화 클래스의 수가 많아진다
  - 다른 요인에 의해 LoginEvent 클래스가 변경되면 LoginSerializer도 변경되어야 한다
  - 여러 이벤트에 대해 표준화를 진행하기 어려워진다.

```Python
# 직렬화를 담당하는 클래스
class EventSerializer:
    def __init__(self, serialization_fields):
        self.serialization_fields = serialization_fields

    def serialize(self, event):
        return {
            field: transformation(getattr(event, field))
            for field, transformation in self.serialization_fields.items()
        }

# 클래스에 serialize 메서드를 제공하는 데코레이터
class Serialization:
    def __init__(self, **transformations):
        self.serializer = EventSerializer(transformations)
    def __call__(self, event_class):
        # 데코레이터 작동하는 부분
        @wraps(event_class)
        def serialzie_method(event_instance):
            return self.serializer.serialize(event_instance)

        event_class.serialize = serialzie_method
        return event_class

def show_original(event_field):
    return event_field

@Serialization(
    username=show_original
)
class LoginEvent:
  #생략

x = LoginEvent()
x.serialize()
```

- 위 코드는 다음과 같은 순서로 작동한다
  - Serialization 클래스가 생성된다(init).
  - x(LoginEvent) 객체를 생성한다(call)
  - LoginEvent를 상태로 가지는 데코레이터를 생성한다. 이때 serialize 메서드가 LoginEvent 객체에 추가된다

- 데코레이터는 중첩해서 쌓을 수 있다. 아래에 있는 것이 먼저 적용된다  .

- - -

인자값을 갖는 데코레이터
========

- arguement를 갖는 데코레이터를 만들고 싶으면 중첩 데코레이터 혹은 데코레이터 객체를 이용한다

중첩 데코레이터
-------------

```py
def sample_dec(s):
    def dec(func):
        @wraps(func)
        def inner():
            print("before", s)
            func()
            print("after", s)
        return inner
    return dec


@sample_dec("1")
def test_f1():
    print("inner")


@sample_dec("2")
def test_f2():
    print("inner")


if __name__ == '__main__':
    test_f1()
    test_f2()

""" result
before 1
inner
after 1
before 2
inner
after 2
"""
```

데코레이터 객체
----------

- 위의 코드는 데코레이터 객체를 사용하였다.
- 데코레이터 객체를 이용하면 3단계 중첩을 쌓는 함수 데코레이터보다 더 간단하게 패러미터를 받는 데코레이터를 구성할 수 있다.

```Python
class DecoratorClass:
    def __init__(self, val1, val2):
        self.val1 = val1
        self.val2 = val2

    def __call__(self, ops):
        @wraps(ops)
        def wrapped(*args, **kwargs):
            print(f'{val1}, {val2}')
            return ops(*args, **kwargs)
        return wrapped


@DecoratorClass(val1 = 1, val2 = 2)
def print_hello():
    print("hi")

print_hello()

# result
1, 2
hi
```

실수
----

- 데코레이터를 만들 때는 흔히 하는 실수들이 있다.
- 원본 함수의 프로퍼티나 속성을 유지하지 않는다: **@wraps(원본)** 를 통해서 래핑.
- 데코레이터에서 원하는 로직은 가장 안쪽에서 수행되어야 한다

```Python
def print_time(function):
    print(LocalDatetime.now())

    @wraps(function)
    def wrapped(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapped

@print_time
def inner():
    #생략

x = inner()
```

- 위의 코드에서 print 부분은 함수를 호출하는 부분이 아닌 import 시에 실행된다
  - decorator를 적용할 때 print가 실행되기 때문
  - 이렇게 되면 의도치 않은 결과를 발생시키므로 print를 wrapped 내부로 이동해야한다

- 물론 위의 방식이 항상 문제인 것은 아니고 아래와 같이 의도적으로 이용할 수도 있다

```Python
REGISTRY = []

def register(event_cls):
    REGISTRY[event_cls.__name__] = event_cls
    return event_cls

@register
class C1:

@register
class C2:
```

- 데코레이터를 사용하면 오히려 복잡도가 증가하여 데코레이터의 내부를 확인해야하는 경우가 발생하면 좋지 못한 데코레이터 이용이다. 데코레이터는 패턴이 명확하여 중복 사용되는 경우에만 최소한으로 사용하도록 하자.

- - -

참고 자료
=====

- [@wraps](https://velog.io/@doondoony/python-functools-wraps)
