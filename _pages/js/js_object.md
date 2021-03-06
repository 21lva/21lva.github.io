---
layout: post
title:  "객체"
date:   2019-05-15 01:02:59
author: Inhyuk
categories: Javascript
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_object.md
---


- 자바스크립트의 객체지향언어는 c++과 상당히 다르다.(다른 대부분의 언어와도 다르다)
- *prototype-based programming* 에 속한다
- *OOP(object oriented programming)* 은 상태와 행동으로 객체라는 것을 만드는 프로그래밍 패러다임이다(캡슐화, 상속, 추상화, 다형성)

- - -

문법
===

- 객체는 연관된 변수와 함수를 캡슐화 한 것이다
- 이 때의 변수를 *프로퍼티(property)* , 함수를 *메소드(method)* 라고 부른다.


1) 생성
-----

```js
var obj1={};
obj1.name="Peter";
obj1.introduce=function(){
  return "this object's name is "+this.name;
}
document.write(obj1.introduce());

var obj2={
 name="Peter",
 introduce=function(){
  return "this object's name is "+this.name;
  }
}
document.write(obj2.introduce());
```


- 첫 방법은 객체 리터럴을 만들고 메소드와 프로퍼티를 추가한다.
- 위의 방법은 객체를 하나 만들 때마다 하나씩 만들어낸다. 이는 중복성을 증가시킨다.
- 두 방식 모두 **JSON(JavaScript Object Notation)** 방식이다. 아래의 방식도 같다

```js
var obj1 = new Object();
obj1.name="Peter";
```

2) 생성자
-----

- 객체를 만드는 함수
- *new 생성자* 는 객체를 생성함.
- 다른 언어에서는 클래스에 생성자가 속해있었지만, 자바스크립트는 그냥 함수가 생성자이다.
- 생성자를 이용하여 프로퍼티와 메소드를 정의하는 것을 *초기화* 라고 한다.

```js
function obj3(name){
  this.name=name;
  this.introduce= function(){
    return "this object's name is "+this.name;
  }
}
var p1 = new obj3("John");
var p2 = obj3("john");//아무일도 일어나지 않는다.
document.write(p1.introduce());
```


- 아래와 같은 방법으로 보통 만든다.

```js
var obj1 = {"key1":10,"key2":"hi"};
var obj2 = {};
var obj3 = new Object();
obj2["key1"]=22;
obj2["key2"]="hello";
```

3) 접근
---

```js
var obj1 = {"key1":10,"key2":"hi"};
for(key in obj1){
  document.write(key+" "+(obj1[key]));
}
```

4) 메소드
-----
- 객체에 정의 되어 있는 함수를 의미
- 객체 안에 value값으로 객체를 내포 가능
- 메소드에서 객체에 접근하려면  *this* 를 사용한다.
- *객체["키값"]* 또는 *객체.키값* 로 호출한다.
- 키는 객체 내부에서 변수 같은 역할을 한다(value는 속성이라고 불림)

```js
var obj={
  "inner_obj" : {
    "key1":1,
    "key2":2
  },
  "func1":function(){
    document.write(this.inner_obj.key1+"<br>");
  }
}
obj.func1();
obj["func1"]();
```

- - -

전역객체
=======

- 모든 객체는 이 **전역객체(window)** 의 프로퍼티이다.
- 함수를 호출할 때 *window.func()* 와 *func()* 는 같다. 즉, 암묵적으로 객체를 명시하지 않으면 *window* 객체에서 불러낸다.
- **node.js** 에서는 *window* 라는 전역 객체가 없다.(*global* 을 씀)
- 환경에 따라 전역객체의 이름이 다르다.

- - -

this
====

- 함수 내에서 *맥락(context)* 을 의미
- 함수 호출의 방법에 따라 this가 가리키는 대상이 달라진다

```js
function if_window(){
  if(window===this)document.write("window is this"+"<br>");
  else document.write("window is not this"+"<br>");
}

if_window();
if_window.apply({});
```


- 첫번째는 함수가 그냥 호출 되므로 전역객체인 window가 호출하는 것이다
- 두번째는 리터럴객체의 메소드로 함수를 추가하고 그 메소드를 호출하는 것이다
- 생성자는 빈 객체를 만들고 *this* 는 만들어진 이 객체를 가리킨다.
- 생성자 생성 -> 할당 순으로 이루어지기 때문에 객체에 어떤 값을 저장해주려면 꼭 *this* 를 사용해야 한다.
- 맥락에 따라 함수(객체)가 어디에(객체,this) 속하느냐가 달라짐(this가 변함)

- - -

리터럴
=====
- 리터럴 : 모든 데이터 타입에 들어가는 데이터값(그냥 의미있는 데이터라고 생각하면 편할듯)
- 함수 리터럴을 변수에 저장하는 것이 함수 표현식이다.
- 리터럴 방식: 선언과 동시에 정의를 하는 방식

```js
var obj1 = {};//리터럴 방식
var obj2 = new Object():
```

- 두 방식 모두 같은 결과를 의미한다(String의 경우에는 다르다)
- 리터럴 방식을 선호하는 이유 : 속도, 가독성, 오버라이딩에 따른 문제 방지

- - -

프로토타입
========

- 자바스크립트는 클래스대신 객체를 복사하여 새로운 객체를 생성하는 프로토타입 언어(ECMA6에서 클래스가 추가되긴 했다)
- *prototype* : 객체의 은닉 속성. 그 객채의 프로토타입이 되는 객체를 가리킨다. 대상이 없으면 null 사용.
- prototype link와 prototype object가 존재

1) Prototype object
----------------

- 객체는 언제나 함수로 생성(*Object(), Function(), Array()*)
- 함수 생성(정의)시 일어나는 일
1. 생성자 자격이 부여된다(*new* 키워드 사용하여 객체 생성 가능)
2. Prototype object 생성 : Prototype Object는 함수에서 **prototype** 이라는 속성을 통해서 접근 가능. **constructor**(함수를 가리킴)와 **__proto__**(prototype link)의 두 속성을 가진다.
- Prototype object도 일반적인 객체이므로 속성을 추가/제거 가능

2) Prototype Link
--------------

- 모든 객체는 **__proto__** 속성을 가지고 있음
- 특정 함수로 만들어진 객체에서 그 함수의 *prototype object* 가 가지는 속성에 접근 가능(프로토타입 체인)
- **__proto__** 는 객체가 생성될 때 사용한 함수의 *protoype object* 를 가리킨다.
- 객체에서 속성을 호출하면, 현재에 존재하는지 찾고 없으면 그 상위 프로토타입을 검색한다. 찾을 때 까지 혹은 차상위 까지 도달하면 그만둔다.

3) 정리
---
- 함수는 특정 속성을 가지는 객체이다.
- 모든 함수는 생성시 constructor 와 prototype(prototype object 의미) 속성을 부여 받는다.
- 함수를 통해서 객체 생성시 **__proto__** 속성이 생기고 이는 함수의 prototype object를 가리킨다.
- 모든 객체의 **__proto__** 를 타고 올라가면 Object함수의 prototype object에 도달한다.


![prototype]({{site.baseurl}}/post_img/{{page.name}}/prototype.png)
- - -

상속
===

추후 추가 예정

- - -

표준 내장 객체
============

- 자바스크립에 built-in 되어 있는 객체(+ 함수)
- *Object, Function, Array, String, Boolean, Number ,Math, Date, RegExp* 가 존재
- 내장 함수의 prototype에 메소드를 추가하면 그 함수로 만든 객체들은 그 메소드를 사용 가능

```js
Array.prototype.rand = function(){
  var index = Math.floor(this.length*Math.random());
  return this[index];
}
var arr = new Array("hi","hello","bye");
document.write(arr.rand());
```


Object
------

- 가장 기본적인 객체. 모든 객체는 Object 생성자 함수를 통해서 만든 인스턴스이다
- Object는 기본적인 만큼 확정하여 사용하지 말자. 모든 객체에 영향을 주기 때문이다.

- - -

데이터 타입
=========

- 데이터 타입에는 객체가 아닌 자료형(원시자료형=기본 데이터 타입), 객체인 자료형(참조데이터타입) 두 종류가 존재한다
- 원시자료형 : number, string, boolean, null, undefined
- 문자열은 원시자료형이지만 메소드 혹은 프로퍼티를 호출하면, 자바스크립트가 임시로 객체로 만들고(래퍼객체) 사용 후 제거함.
- String, Number, Boolean => 래퍼객체

- - -

복제와 참조
=========

- 복제 :어떤 데이터의 값을 복제하여 다른 데이터에 저장하는 것

참조(reference)
--------------

- 어떤 데이터 자체를 저장하는 것이다(해당 데이터가 저장된 주소를 저장한다)
- 원시자료형이 아닌 자료형은 참조 자료형(복제되지 않고 참조한다)
- 함수의 인자로 참조자료형이 주어지는 경우에도 복제가 되지 않고 참조를 하므로 함수 내부에서 변화 시킬 수 있다.

```js
//원시 자료형은 복사한다
var primitive1 = 1;
var primitive2 = primitive1;
primitive2=3;
document.write(primitive1+"<br>");

//참조 자료형은 참조한다
var x = {id:1};
var y = x;
y.id=2;
document.write(x.id+"<br>");

// 함수에 참조자료형의 인자를 제공하면 복사하지 않고 참조한다
var x = {id:1};
function change_to_3(d){
  d.id=3;
}
change_to_3(x);
document.write(x.id);

```



- - -

캡슐화
=====

- 자바스크립트는 **private** 와 **public** 키워드를 지원하지 않는다
- **this.프로퍼티** 가 아닌 **var 프로퍼티** 로 프로퍼티를 만들면 외부에서 접근 불가능 하다.(public 메서드가 클로저 역할을 한다)
- 이 방식을 **모듈 패턴** 이라고 한다.

```js
funtion Clas(arg){
  var privateP = arg;
  this.getPrivateP = funtion(){
    return privateP;
  }
}
```

- 위의 방식을 아래 처럼 함수를 호출하면 객체를 반환하는 방식으로 만들 수 있다(더 자주 쓰인다)

```js
funtion Clas(arg){
  var privateP = arg;
  return{
    getPrivateP = funtion(){
      return privateP;
    }  
  }
}
var obj = Clas(2);
```

- 위의 방법은 private 프로퍼티로 객체나 배열을 사용하면 얕은 복사(참조)를 하게 되고, 사용자가 쉽게 위의 private 프로퍼티에 접근 가능하다. -> 깊은 복사를 하도록 하자.


- 생성자를 만드는 방식이 아니기 때문에 객체(cls)의 프로토타입에 접근이 불가능하다. 이 경우 모든 메서드 프로토타입을 새로 생기는 객체(obj)에 포함해야 한다.
- 객체를 반환하는 것이 아닌 함수를 반환해보자

```js
var Clas = function(){
  var privateP;
  var F = function(arg){
    privateP=arg;
  }
  F.prototype={
    getPrivateP:function(){
      return privateP;
    }
  };
  return F;
}();

var obj = Clas(2);
```
