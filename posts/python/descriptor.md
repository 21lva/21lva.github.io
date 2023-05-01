---
layout: post
title:  "Descriptor"
date:   2022-06-17 02:02:59
author: Inhyuk
category: Python
tags: Python
cover:  "/assets/instacode.png"
name: descriptor.md
---

Descriptor
==========

- 디스크립터는 2가지로 이루어진다.  
  1. client class: 디스크립터 기능을 활용. 디스크립터 객체를 클래스 속성(인스턴스 속성이 아님)으로 갖고 있어야함.
  2. descriptor class: 디스크립터 로직을 갖는 클래스. *__get__, __set__, __delete__, __set_name__*중 하나 이상이 구현되어야 함.
- 디스크립터는 객체이기 때문에 selffㅡㄹ 첫 번째 parameter로 사용.


__get__(self, instance ,owner)
------------------

- arguements
  - self: descriptor 객체 자신
  - instance: descriptor를 소유하고 있는 객체
  - owner: instance에 주어지는 객체의 클래스. 클라이언트 클래스에서 디스크립터를 직접 사용하는 경우에 사용할 parameter


```py
class Descriptor:
    def __get__(self, instance, owner):
        if instance is None:
            #보통은 self를 그대로 반환한다.
            return f"{self.__class__.__name__}.{owner.__name__}"
        return f"value for {instance}"


class ClientClass:
    descriptor = Descriptor()

if __name__ == '__main__':
    print(ClientClass.descriptor)
    print(ClientClass().descriptor)

"""result

Descriptor.ClientClass
value for <__main__.ClientClass object at 0x100f014f0>
"""
```

__set__name__(self, owner, name)
-----------------

- 디스크립터는 디스크립터가 다루려는 객체 속성의 이름을 알아야한다. __get__, __set__ 에서 사용된다.
- 3.6버전이전에서는 객체 초기화시에 __set__name__을 사용하지 않고 직접 값을 전달해야 했지만, 3.6 이상에서는 __set__name__을 정의해놓으면 자동으로 할당이 된다.

```py
class Descriptor: #descriptor
    def __init__(self):
        self._name = None

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name] # 저장된 값을 가져오기 위함

    def __set_name__(self, owner, name):
        print(name)
        self._name = name

    def __set__(self, instance, value):
        instance.__dict__[self._name] = value


class Client:
    value_1 = Descriptor()
    value_2 = Descriptor()


if __name__ == '__main__':
    cl1 = Client()
    cl1.value_1 = "11"
    cl1.value_2 = "22"
    cl2 = Client()
    cl2.value_1 = "aa"
    cl2.value_2 = "bb"
    print(cl1.__dict__)
    print(cl2.__dict__)

""" result
value_1
value_2
{'value_1': '11', 'value_2': '22'}
{'value_1': 'aa', 'value_2': 'bb'}
"""
```

__set__(selft, instance, value)
----------------

- arguements
  - self: descriptor 객체 자신
  - instance: descriptor를 소유하고 있는 객체
  - value: descriptor에 할당하려는 값
- set은 유효성 검사할 때 유용하게 사용가능

```py
class Validation:
    def __init__(self, valid_func, erro_msg: str=""):
        self.valid_func = valid_func
        self.error_msg = erro_msg

    def __call__(self, value):
        if not self.valid_func(value):
            raise ValueError(f"[Value Error({value})]: {self.error_msg}")


class Field: #descriptor
    def __init__(self, *validations):
        self.validations = validations
        self._name = None

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name] # 저장된 값을 가져오기 위함

    def __set__(self, instance, value):
        for valid in self.validations:
            valid(value)
        instance.__dict__[self._name] = value #값 저장. setattr를 사용하면 무한 루프에 빠지기 때문에 객체의 딕셔너리에 직접 접근해야 한다.

    def __set_name__(self, owner, name):
        print(name)
        self._name = name


class Client:
    int_bigger_than_10 = Field(Validation(lambda x: x > 10, "Should be bigger than 10"))
    int_smaller_than_10 = Field(Validation(lambda x: x < 10, "Should be smaller than 10"))
    int_in_1_10 = Field(
        Validation(lambda x: x <= 10, "Should be smaller than 10"),
        Validation(lambda x: x >= 1, "Should be smaller than 10"),
    )
```

__delete__(self, instance)
------------

- **del client.descriptor**시에 호출되는 메서드
- 특정 경우이거나, 권한이 있을 때만 객체를 삭제할 수 있도록 로직을 구현할 수 있다.

- - -

종류
===

- 디스크립터는 2가지 종류가 있다.
  1. 데이터 디스크립터: __set__ 또는 __delete__를 구현. 클라리언트 객체의 사전에서 같은 이름의 다른 데이터보다 먼저 사용된다.
  2. 비데이터 디스크립터: __set__과 __delete__를 미구현 + __get__을 구현. 클라이언트 객체의 사전에 같은 이름을 갖는 다른 데이터가 있으면 그 데이터가 우선시된다.

- - -

응용
===

중복 로직 지양을 위한 디스크립터
-----------

- 실제 디스크립터를 사용하는 한 예를 알아보자. 어떤 클래스의 속성 값이 변할 때마다 그 값을 검사하는 로직을 추가하려 한다고 가정하자. 기존에는 **@property**를 이용하여 구현하였다.

```py
class C1:
    def __init__(self):
        self._name = None

    @property
    def name(self):
        if self._name is None:
            raise ValueError("Name is not set yet.")
        return self._name

    @name.setter
    def name(self, name):
        assert 3 <= len(name) <= 5
        self._name = name


if __name__ == '__main__':
    c = C1()
    c.name = "abc"
    print(c.name)
    c.name = "aaaaaaaaaaaaaaaaaaaaa"
```

- 위의 코드에는 *assert*를 통해서 setter에 validation을 추가하였다. 만약에 해당 로직을 사용하는 곳이 많다면 할 수 있는 방법은 4가지가 있다.
  1. 단순히 코드 복사 or validation 함수(또는 클래스) 이용: 코드 복사는 지양해야하며, validation함수를 이용하면 누락되는 경우 발생.
  2. property builder
  3. decorator
  4. descriptor

- 4번의 데코레이터를 이용하는 방법은 다음과 같이 할 수 있다

```py
class BoundedName:
    def __init__(self):
        self._name = None

    def __set_name__(self, owner, name):
        self._name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name]

    def __set__(self, instance, value):
        assert type(value) == str
        assert 3 <= len(value) <= 5
        instance.__dict__[self._name] = value


class C1:
    name = BoundedName()


if __name__ == '__main__':
    c = C1()
    c.name = "abc"
    print(c.name)
    c.name = "aaaaaaaaaaaaaaaaaaaaa" #오류 발생!!
```

클래스 데코레이터 대체
----------------
