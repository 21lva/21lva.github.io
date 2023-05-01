---
layout: post
title:  "반복문, 제어문"
date:   2019-08-10 02:02:59
author: Inhyuk
category: kotlin
tags: kotlin
cover:  "/assets/instacode.png"
name: cont.md
---


when
====

- java의 switch와 비슷

```
when(x){
  1,2 -> print("1 or 2")
  3,4 -> print("3 or 4")
  else -> {
    print("NOTHING")
  }
}

//반환값 사용
when(x){
  isZero(x) -> print("zero")
  else -> {
    print("NOTHING")
  }
}

//범위 지정 가능
when(x){
  in 1..10 -> print("less than 11")
  !in 10..20->print("more than 20")
  else -> print("other")
}

//is 사용
val resut = when(x){
  is String -> "String"
  else -> "not String"
}
```

- - -

for
====

```
//이상,이하
for(i in 1..10)

//내림
for(i in 10 downTo 1)

//step
for(i in 10 downTo 1 step 2)
```

- - -

return
=====

- 람다식에서 return은 사용 불가능(inline에서는 비지역 반환으로 사용 가능)
- 람다를 종료하고자 하면 라벨을 사용(**return@**)

```
//라벨 이름 지정
fun main(){
   println("Before outter")
   outerF label@{
       println("This is a lambda")
       return@label
   }
   println("After outter")
}


fun outerF(lambda:()->Unit){
   println("Before lambda")
   lambda()
   println("After lambda")
}

//결과
Before outter
Before lambda
This is a lambda
After lambda
After outter
```

- 다음과 같이 암묵적인 라벨을 사용할 수도 있다

```
fun main(){
    println("Before outter")
    outerF {
        println("This is a lambda")
        return@outerF
    }
    println("After outter")
}


fun outerF(lambda:()->Unit){
    println("Before lambda")
    lambda()
    println("After lambda")
}

//결과
Before outter
Before lambda
This is a lambda
After lambda
After outter
```

- 람다를 변수에 저장해서 사용하려면 다음과 같이 한다

```
fun main(){
    val lambda :() -> Unit = lit@ {
        println("This is a lambda")
        return@lit
    }
    println("Before outter")
    outerF(lambda)
    println("After outter")
}


fun outerF(lambda:()->Unit){
    println("Before lambda")
    lambda()
    println("After lambda")
}

//결과
Before outter
Before lambda
This is a lambda
After lambda
After outter
```
