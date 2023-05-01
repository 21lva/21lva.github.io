---
layout: post
title:  "위임"
date:   2019-08-10 02:02:59
author: Inhyuk
category: kotlin
tags: kotlin
cover:  "/assets/instacode.png"
name: delegation.md
---

- delegation pattern: 오브젝트들을 합성하는 패턴. 호출된 object가 delegate라는 helper object에게 요청을 위임해서 요청을 처리

위임
===

- 상속은 강력하지만 코드 재사용만을 위해서 사용하면 문제점이 발생한다. 불필요한 메서드, 필드 등을 상속하게 된다. 즉, 부모와 자식클래스간의 불필요한 의존성이 생기게 된다. kotlin에서는 상속을 최소화하기 위해서 클래스가 기본으로 *final*로 생성된다
- 상속과 대비되는 위임의 장점은 유연함이다. 메서드 호출 하나ㄹ를 다른 오브젝트에게 전달하기 때문에 합성이라고 생각할 수 있다.
- 코틀린에서 위임의 방법은 2가지가 있다: 속성위임, 클래스 위임


property 위임
---------

- getter, setter의 호출을 delegate에게 전달한다. *by* 키워드 뒤에 오는 것이 *delegate(위임 객체)*이다

```
class A {
  var value: String by Delegate()
}

//위는 아래와 같다. 이 과정은 컴파일러가 대신해준다
class A {
  private val deletgate = Delegate()
  var value: String
  set(value: String) = delegate.setValue(..., value)
  get() = delegate.getValue()
}
```

#### lazy

- *lateinit*은 *var* 타입의 필드에만 작동한다. *val*은 **lazy**를 사용해야한다

```kt
val lazyValue: String by lazy {
  println("init lazy value")
  "abc"
}
```

#### Delegates.vetoable

- 값 변경을 거부할 수 있게 해주는 표준 delegate
- true이면 값 변경, 아니면 값 변경하지 않음

```kt
fun main(args: Array<String>) {
    val x = X()
    println(x.value)
    x.value=1
    println(x.value)
    x.value=3
    println(x.value)
}

class X{
    var value: Int by Delegates.vetoable(0){ property, oldValue, newValue ->
        newValue == oldValue + 1
    }
}

//result
0
1
1
```

#### map deletgate

- 함수 or class의 생성자에 여러 parameter대신에 하나의 map을 제공하여 초기화할 수 있게 해준다
- key의 이름이 속성명과 동일해야 하며, 전달받지 않은 속성에 접근하려하면 **NoSuchElementException**이 발생한다

```kt
fun main(args: Array<String>) {
    val u = User(mapOf("name" to "Bob", "age" to 12))
    println(u)
}

data class User(val params: Map<String, Any?>){
    val name: String by params
    val age: Int by params
}

//result
User(params={name=Bob, age=12})
```

#### Custom delegate

- 사용자가 직접 delegate를 만들 수도 있다.
- **ReadWriteProperty**를 상속받아서 만들 수 있다.

```
fun main(args: Array<String>) {
    val x = X()
    println(x.value)
    x.value = "455"
    println(x.value)
}
class X{
    var value: String by CustomDel("123")
}

class CustomDel(initVal: String): ReadWriteProperty<Any?, String>{
    private var value = initVal

    override fun getValue(thisRef: Any?, property: KProperty<*>): String {
        println("GET")
        return this.value
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("SET VALUE as $value")
        this.value = value
    }
}

//result
GET
123
SET VALUE as 455
GET
455
```

클래스 위임
------------

- A,B가 같은 인터페이스를 implement 하는 경우에 A 대신 B가 역할을 수행하게 가능. 이때 A가 B에게 위임했다고 한다
- 클래스 위임을 이용하면 데코레이터 패턴도 쉽게 만들 수 있다
- 위임으로 처리하고 싶지 않은 메서드는 *override*하면 된다

```
fun main(args: Array<String>) {
    val x = X()
    X().printing()
    Y().printing()
    Z(X()).printing()
    Z(Y()).printing()
}
interface I {
    fun printing()
}

class X: I{
    override fun printing() {
        println("X")
    }
}

class Y: I by X()
class Z(val i: I): I by i

//result
X
X
X
X
```
