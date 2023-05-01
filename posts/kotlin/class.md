---
layout: post
title:  "클래스"
date:   2019-08-10 02:02:59
author: Inhyuk
category: kotlin
tags: kotlin
cover:  "/assets/instacode.png"
name: class.md
---


선언
===

```
class ClassName{

  var v1 : String
  val v2 : String

  fun f1(){
    ...
  }
}

//내용이 비어 있으면 중괄호 생략 가능
class ClassName


fun main(){
  val ob1 = ClassName()
  ob1.v1 = "hello"
  ob1.v2 = "hello"//val이라 불가능
  ob1.f1()
}
```

- 프로퍼티 : 필드(멤버변수) + 접근메서드

- - -

생성자
====

주 생성자
-------

```
class MyC constructor(_v1:String ="hi", _v2 : Int = 4){
  var v1 = _v1
  val v2 : Int // type 지정해줘야 함
  init{
    if(v1=="hi"){
      v2 = _v2
    } else {
      v2 = _v2+1
    }
  }
}

//constructor 생략 가능
class MyC(_v1:String ="hi", _v2 : Int = 4){
  var v1 = _v1
  val v2 : Int // type 지정해줘야 함
  init{
    if(v1=="hi"){
      v2 = _v2
    } else {
      v2 = _v2+1
    }
  }
}

//constructor에 프로퍼티 정의 가능
class MyC(var v1:String ="hi", val v2 : Int = 4){
}
```

부 생성자
-------

- 코틀린에서는 주생성자와 부생성자가 구별되고 주 생성자는 하나만 가능하지만 부 생성자는 여러 개 가능
- 부 생성자는 먼저 주 생성자를 호출한다(주 생성자가 있으면)

```
fun main(){
    val ob1 = MyC(v2=3)//Primary constructor
    val ob2 = MyC("hello",v2=3L)//Secondary constructor
    println(ob2.v1)
}

class MyC(var v1:String ="hi", val v2 : Int){
    init{
        println("init")
    }

    constructor(v1 : String, v2 : Long) : this(v2 = 3) {
        println("Secondary Constructor")
        this.v1 = v1
    }
}

//결과
init
init
Secondary Constructor
hello
```

- - -

상속
===

- 모든 클래스는 Any 클래스의 sub class
- 자바와 달리 모든 클래스는 상속 불가능한 클래스이다(상속하려면 클래스에 **open**가 붙어야함)

오버라이딩
--------

- super class에서 **open** 클래스로 sub class에 *override**를 붙여서 사용한다
- override를 명시적으로 막으려면 **final**을 사용한다

```
open class Bird(var name: String){
  open fun sing() = println("Sing")
  final fun finalSing(){
    println("This method cannot be overrided")
  }
}

class SubBird(name: String//super class에 정의된 프로퍼티
  val lang :String ="koKR"//서브 클래스에서 새로 생기는 프로퍼티는 새로 정의 한다
  ):super(name){//super class의 생성자 호출
    override fun sing(){
      ...
    }
  }
```

- - -

Companion Object
========

- 자바에서 사용하는 **static method, variable**을 사용하기 위해서는 **companion object**를 사용한다

```
fun main(){
    MyC.callStaticMethod()
    MyC.f1() //Error
}

class MyC(){
    companion object{
        fun callStaticMethod() = println("This is a static method")
    }
    fun f1(){
        println("This is a member method")
    }
}
```

- - -

내부 클래스
=======

- 코틀린에는 2개의 내부 클래스가 존재
  - Nested: 자바의 static 내부 클래스와 동일. 객체 생성 없이 사용 가능
  - Inner: 자바의 Member 내부 클래스와 동일: inner 키워드 필요. 객체 생성 가능

Inner class
----------

- Inner class: 클래스 내부에 선언하는 클래스. 바깥 클래스를 통해서만 사용 가능. 바깥 클래스에 접근 가능

```
fun main(){
    OutC().InC().inF() //outter를 통해서 inner 접근
}

class OutC(val outterProperty:String="out"){
    fun outF(){
        println("Out fun2 called")
    }

    inner class InC(){
        fun inF(){
            println("Inner Func")
            outF()//접근가능
        }
    }
}

//결과
Inner Func
Out fun2 called
```

Nested class
-------------

- - -

인터페이스
========

- 인터페이스는 기본적으로 **open**이다

```
open class A{
  ...
}

interface B{
  ...
}

class C: A(), B{//interface는 생성자가 없기 때문에 생성자를 호출하지 않는다
  ...
}
```

- - -

접근자
=====

- private(-): 클래스의 외부에서 접근 불가능
- protected(#): sub class에서 접근 가능
- internal(~): 같은 모듈 내부에서 접근 가능
- public(기본,+): 모든 접근 허용

- - -

지연 초기화
=========

- 지연 초기화: 프로퍼티 초기화를 프로퍼티가 접근될때 수행된다
- lateinit: **var** 을 지연 초기화
- 클래스가 특정 클래스에 의존적이고 그 클래스가 생성되지 않았을 때 사용 가능

```
fun main(){
    var x = OutC()
    x.f()
    x.name = "hello"
    x.f()
}

class OutC(){
    lateinit var name : String
    fun f() = println(
        if(::name.isInitialized) "init : $name " else "not init"
    )
}

//결과
not init
init : hello
```

double colon
------------

- [참고 자료1](https://medium.com/harrythegreat/%EC%BD%94%ED%8B%80%EB%A6%B0%EC%9D%98-%EB%8D%94%EB%B8%94%EC%BD%9C%EB%A1%A0-%EC%B0%B8%EC%A1%B0-73ff25484586)
- [참고 자료2](https://androidtest.tistory.com/105)
- **::** : 변수를 객체로 접근하여 사용
- 리플렉션: 런타임에 객체, 함수, 프로퍼티 등을 분석하는 방법

lazy
-----

- **val** 로 설정된 프로퍼티를 지연초기화 하는데에 사용됨

```
fun main(){
    var x = OutC()
    x.f2()
    x.f()
}

class OutC(){
    val v1 by lazy{
        println("Lazy init")
        "val1"
    }

    fun f2() = println("Before init")
    fun f() = println("V1: $v1")
}

//결과
Before init
Lazy init
V1: val1
```

- - -

object
=====

- object는 **singleton** 객체, 익명 객체를 만들 때 사용

```
fun main(){
    Ob.sayName()
    Ob.name = "new name"
    Ob.sayName()
}

object Ob{
    var name = "name1"
    fun sayName() = println("My name is $name")
}

//결과
My name is name1
My name is new name
```

익명 객체
------

- 익명객체: 이름이 없이 한 번 생성되고 사용되는 객체

```
fun main(){
    val x1 = Mc()
    val x2 = Mc()
    x1.obj.sayHi()
    x2.obj.sayHi()
    x1.obj.sayHi()
    x2.obj.sayHi()
}

interface InF{
    fun sayHi() = println("hi")
}

class Mc{
    val obj = object: InF{
        var name = "name"
        override fun sayHi() {
            println("hello $name")
            name = "name2"
        }
    }
}
```

- - -

추상 클래스
========

- 인터페이스와 달리 프로퍼티 설정 가능(메서드는 둘 다 설정 가능)

```
fun main(){
    //val x = Ab()//에러
    val y = X()
    y.f()
    y.g()
    y.h()
}

abstract class Ab{
    val x = 12
    abstract val prop : String
    abstract fun f() //abstract는 메서드를 정의할 수 없다
    fun g() = println("hello") //sub class에서 override 불가
    open fun h() = println("hello")
}

class X : Ab(){
    override val prop = "prop"//추상 프로퍼티
    override fun f() {
        println("f of X")
    }

    override fun h() {
        println("h of X")
    }
}

//result
f of X
hello
h of X
```

- - -

데이터 클래스
=========

- 데이터를 저장하고 전달하기 위한 클래스
- getter/setter 제공
- 비교를 위한 equals(), hashCode() 제공
- 프로퍼티를 문자열로 변환해주는 toString() 제공
- 객체 복사를 위한 copy() 제공
- abstract, open, sealed, inner 사용 불가
- 주 생성자의 모든 parameter는 val, var이다(상속 불가능)
- 최소 하나의 parameter를 가지고 있어야함
- 부생성자, init 사용 가능

```
fun main(){
    val book1 = Book("hi", 100)
    val book2 = Book("hi", 100)
    val book3 = Book("hello", 100)
    println(book1 == book2)
    println(book1.equals(book2))//위 연산과 동일
    println(book1.hashCode()==book2.hashCode()) //데이터가 같으면 같은 객체로 취급
    println(book1.equals(book3))
    println(book1.toString())
}

data class Book(val name : String, var price:Int){
    constructor(name: String, price: Int, priceDown: Boolean):this(name,price){
        this.price = if(priceDown) price-1 else price
    }
}

//결과
true
true
true
false
Book(name=hi, price=100)
```

Destruct
--------

- 객체가 가지고 있는 프로퍼티를 개별 변수로 분해하는 것

```
fun main(){
    val book1 = Book("hi", 100)

    val (name, price) = book1
    val (_, price2) = book1 //언더스코어를 이용한 무시
    println(name)
    println(price)
    println(price2)
}

data class Book(val name : String, var price:Int){
    constructor(name: String, price: Int, priceDown: Boolean):this(name,price){
        this.price = if(priceDown) price-1 else price
    }
}

//결과
hi
100
100
```

- - -
sealed vs enum
==============

- [비교](https://blog.kotlin-academy.com/enum-vs-sealed-class-which-one-to-choose-dc92ce7a4df5)

sealed class
------------

- 미리 만들어 놓은 자료형들을 묶어서 제공하기 위한 열거형 클래스의 확장형
- 모든 생성자는 private(객체 생성 불가)
- 같은 파일내에서만 상속 가능

```
sealed class Expr {
    class Num(val value:Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}


fun eval(e: Expr) : Int =
        when(e) {
            is Expr.Num -> e.value
            is Expr.Sum -> eval(e.right) + eval(e.left)
            //else -> //else 가 필요 없다
        }
```

enum class
---------

- 여러 개의 상수를 선언하고 열거된 값을 조건에 따라 선택할 수 있는 클래스
- 실드 클래스와 달리 다양한 자료형을 다루지는 못함

```
fun main(){
    val w1 = Way.EAST
    val w2 = Way.NORTH

    determine1(w1)
    determine2(w1)
    determine1(w2)
    determine2(w2)
}

fun determine1(w:Way){
    when(w){
        Way.NORTH, Way.SOUTH ->println("NORTH OR SOUTH")
        else -> println("EAST OR WEST")
    }
}

fun determine2(w:Way){
    when(w.n){
        in 1..2 ->println("NORTH OR SOUTH")
        else -> println("EAST OR WEST")
    }
}

enum class Way(val n:Int){
    NORTH(1), SOUTH(2), EAST(3), WEST(4);//semi colon 으로 끝을 표시

    fun sayNum(){
        println("My way is $n")
    }
}

//Result
EAST OR WEST
EAST OR WEST
NORTH OR SOUTH
NORTH OR SOUTH
```
