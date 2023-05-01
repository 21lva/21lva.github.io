---
layout: post
title:  "함수"
date:   2019-08-10 02:02:59
author: Inhyuk
category: kotlin
tags: kotlin
cover:  "/assets/instacode.png"
name: function.md
---

기본
===

```
fun 함수이름(매개변수: 매개변수타입 = 디폴트값) : 반환타입{
  ...
}
```

- 반환타입이 **Unit** 인 경우에는 생략 가능
- Unit vs void(java)
  - void: 아무것도 반환하지 않음
  - Unit : Unit이라는 객체를 반환. kotlin에서 void역할을 함

```
fun sum(a:Int, b:Int ): Int{
  return a+b
}

//함수가 한 줄이면 중괄호와 return 생략 가능
fun sum(a:Int, b:Int):Int = a+b

//return 값도 생략 가능
fun sum(a:Int, b:Int) = a+b
```

매개변수
------

```
함수이름(매개변수1, 매개변수2)
함수이름(매개변수1)//default value가 있는 경우 생갹 가능
함수이름(매개변수1=매개변수1, 매개변수2=매개변수2)

//매개변수의 갯수가 정해져있지 않을때
fun f1(vararg counts: Int){
  ...
}
```

- 매개변수는 **val** 이다

- - -

함수형 프로그래밍
=============

순수 함수
------

- 부작용이 없는 함수
- thread safe하다
- 입력값에 따르다 결과값이 항상 같다
- 함수 외부의 상태를 바꾸지 않는다

일급 객체
------

- 함수의 인자로 전달할 수 있다
- 함수의 반환값에 사용할 수 있다
- 변수에 담을 수 있다
- 고차 함수: 다른 함수를 인자로 사용하거나 함수를 결과값으로 반환하는 함수

람다식
-----

- 일급 함수 중 이름이 없는 함수

```
//람다식 형태
val sum = {x:Int, y:Int -> x+y}

//타입을 미리 지정했다면 람다식에서 생략 가능
val sum : (Int,Int)->Int = {x,y-> x+y}

//함수가 여러 줄이면 마지막 줄이 리턴값의 역할을 함
val sum : (Int,Int)->Int = {x,y ->
  println("x+y")
  x+y
}
```

```
//매개변수가 없는 형태
val sayHi: () -> Unit = {println("hi")}
val sayHi = {println("hi")}

//매개변수가 하나일 경우
val sayHiTo: String -> Unit = {target -> println("hi $target")}

//매개변수가 하나일 경우 매개변수도 생략 가능($it 사용)
val sayHiTo: String -> Unit = {println("hi $it")}

//고차함수에서 람다식 사용
fun high(first: (Int,Int) -> Int, a:Int, b:Int): Int{
  return first(a,b)
}

high(sum,1,2)
```

```
//매개변수 위치가 가장 마지막일 경우
fun high(a:Int, b:Int, first: (Int,Int) -> Int):Int{
  return first(a,b)
}

high(1,2,sum)
high(1,2,{x:Int, y:Int -> x+y})
high(1,2){x:Int, y:Int -> x+y}
```

- 일반 함수는 일급 함수가 아니다.

```
fun sum(x:Int, y:Int) = x+y

//오류
high(1,2,sum)

//됨
high(1,2,::sum)
```

익명 함수
------

```
fun(x:Int, y:Int): Int = x+y

```

- 이름이 없는 일반 함수 (람다함수가 아님)
- 람다와 익명함수 모두 **function literals** 이다(한 번 쓰고 삭제한다는 것)
- vs 람다
  - 둘의 차이는 return의 행동 방식
  - lambda에서 return은 그 람다를 호출한 고차함수를 return함(inline과 비슷하게 행동). lambda를 끝내려면 라벨을 사용함(return@라벨)
  - 익명함수는 그 익명함수를 return함

- - -

인라인 함수
=======

- 컴파일시에 함수 자체를 복사해서 적어놓는 함수
- 짧게 사용하는 함수에서 stack생성에 필요한 overhead를 줄일 수 있음

```
inline fun inlineFunction(){

}
```

- inline 함수에 매개변수로 람다식이 들어가면 람다식도 그대로 복사된다. noinline으로 방지

```
inline fun f1(noinline f2 : () -> Unit){
  ...
}
```

비지역 반환
--------

- 비지역 반환 : 람다 함수내의 return을 이용하여 람다 함수를 포함하는 고차함수를 리턴하게 하는 방법

```
fun main(){
  outF(1){
    println("inner lambda")
    return
  }
  println("After outter fun")
}

inline fun outF(a:Int, inF:()->Unit){
  println("Before")
  inF()
  println("After")
}

//결과
Before
inner lambda
```

- lambda는 inline함수에서 사용하는 경우가 아니면 return 사용불가
- lambda의 return은 그 람다를 사용하는 함수를 리턴
  - inline을 통해서 outF의 값이 그대로 복사 됨 -> 람다의 return을 통해서 main을 반환

- 아래는 원치 않는 비지역 반환을 막는다


```
fun main(){
  outF(1){
    println("inner lambda")
    //crossinline의 경우 return 사용 불가
  }
  println("After outter fun")
}

inline fun outF(a:Int, crossinline inF:()->Unit){
  println("Before")
  inF()
  println("After")
}

//결과
Before
inner lambda
After
After outter fun
```


- - -

확장 함수
======

- 클래스의 멤버 메서드로 되어 있지 않는 함수를 메서드로 포함시키는 것
- 최상위 클래스인 Any에 확장함수를 구현하면 모든 클래스에서 사용가능

```
fun 확장클래스.함수이름(){
  ...
}

//예제
fun main(){
    val x : String = "Bread"
    x.sayHi()
}

fun String.sayHi(){
    println("Hi $this")
}

//결과
Hi Bread
```

- - -

중위 함수
======

- 중위 표현법: 클래스의 멤버를 호출할 때 사용하는 **.** 와 **()** 를 생략하고 직관적으로 함수를 호출하게 하는 방법
- infix 키워드 사용
- 멤버 메서드 혹은 확장함수에만 적용 가능
- 매개변수가 하나여야함

```
fun main(){
    val x : String = "Bread"
    x sayHiTo "Lisa"
}

infix fun String.sayHiTo(target:String){
    println("My name is $this. Hi $target")
}

//결과
My name is Bread. Hi Lisa
```

- - -

Tail recursive function
================

- 재귀함수에서 계산을 먼저하고 재귀함수가 호출되는 형태의 재귀함수
- 일반적인 재귀함수는 재귀함수가 호출되고 계산이 되기 때문에 스택 오버플로우 문제가 발생할 수 있음
- 꼬리 재귀함수를 이용하면 스택오버플로우를 방지할 수 있음(계산한 값을 넘길때 call stack에서 이전 함수를 지워버림)

```
fun main(){
    val x1 = doRec1(150000) // 정상 동작
    val x2 = doRec2(150000) // Stack Overflow Error
    val x3 = doRec3(150000) // Stack Overflow Error
}

tailrec fun doRec1(n:Long,res : Long=0):Long{
  //정상 꼬리 재귀
    return if(n==1L) res else doRec1(n-1, res+n)
}


fun doRec2(n:Long,res : Long=0):Long{
  //꼬리 재귀 형태이지만 tailrec을 사용하지 않았기 때문에 꼬리 재귀 형식으로 동작하지 않음
    return if(n==1L) res else doRec2(n-1, res+n)
}


tailrec fun doRec3(n:Long):Long{
  //tailrec을 추가하였지만 꼬리재귀 형태가 아니기때문에 tailrec무시
    return if(n==1L) n else doRec3(n-1)+n
}
```

- - -

함수와 변수의 범위
=========

최상위 함수
--------

- 최상위 함수 : **kt** 파일내에 그대로 사용되어지는 함수. 선언부의 위치와 상관없이 사용 가능
- 지역함수 : 함수내에 선언된 함수. 선언이 되어야만 사용가능. 지역을 벗어나면 사용 불가능

```
fun main(){
    sayHi()
    localF1() // ERROR!! 지역함수는 선언보다 먼저 사용 불가능
    fun localF1(){
        println("local function 1")
    }
    localF1()
}


fun sayHi(){
    localF1() // ERROR!! 지역함수는 자신을 선언한 함수 밖에서 사용 불가능
   println("hi")
}

```

변수의 범위
------

- 전역변수: 최상위에 존재하는 변수. 프로그램 실행동안 유지됨. 해당 패키지에서 모두 사용 가능
- 지역변수: 해당 함수 블록내에서만 유지
