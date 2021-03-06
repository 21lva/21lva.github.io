---
layout: post
title:  "javascript 2 - 함수"
date:   2019-05-15 01:02:59
author: Inhyuk
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: 2019-05-16-js2.md
---

- 자바스크립트에서 함수도 객체이다(특정 속성을 가지는 객체)
- 객체이기 때문에 **var x = new Function("a","b","return a+b");** 로 할 수 있다.

선언식 vs 표현식
=========



1) 선언식
---------

- 일반적인 프로그래밍 언어에서의 함수선언

```
function function_name(input1, input2){
  // write what do you want to do
  return res;
}

function_name(1,2);
```

2) 표현식
---------

```
var function_name = function (input1,input2 ){
  return res;
}
fuction_name(1,2);
```

- 호이스팅에 영향 받지 않음
- 클로져, 콜백(함수의 매개함수로 사용)으로 사용 가능

- - -

스코프(유효범위)
=============

1) var
------

- var 변수는 함수 스코프를 가지고, let,const는 블록 스코프를 가짐
- 블록 스코프 : *{}* 안에서만 사용 가능
- 함수 스코프 : 변수가 선언된 함수 안에서 사용 가능. 글로벌 스코프의 경우 함수 스코프와 같은 취급. 함수 안에서 씌여진 변수는 함수가 종료될 때 까지 계속 유지(for문의 초기값으로 선언된 변수도 마찬가지)
- 자바스크립트는 중복 선언 허용(내부 변수의 범위를 벗어나면 그 변수가 사라지고 원래 변수가 나타남)
- 함수 호출시 어떤 변수를 함수 내부에서 사용함 -> 함수 내부에 변수가 정의되어 있지 않으면, 함수가 정의된 곳에서 확인함(호출한 곳이 아님)

```
var x =10;
document.write(x+"<br>");
function fun(){
  var x =1;
  document.write(x+"<br>");
  for(var i =0;i<=2;++i){
    var x=i;
  }
  document.write(x+"<br>");  
}
fun();
document.write(x+"<br>");
```
#### result
<script>
var x =10;
document.write(x+"<br>");
function fun(){
  var x =1;
  document.write(x+"<br>");
  for(var i =0;i<=2;++i){
    var x=i;
  }
  document.write(x+"<br>");  
}
fun();
document.write(x+"<br>");
</script>

2) let
-----

- *var* 을 이용하면 스코프를 고려해야함
- *let* 는 블록 스코프(대부분의 프로그래밍언어에서 사용)를 사용
- 아래의 예제는 *let* 으로 선언된 xx가 출력이 안됨

```
<script>
if(true){
  let xx =10;
  var yy =20;  
}
document.write("value of yy is "+yy+"<br>");
document.write("value of xx is "+xx+"<br>");
```
#### result
<script>
if(true){
  let xx =10;
  var yy =20;  
}
document.write("value of yy is "+yy+"<br>");
document.write("value of xx is "+xx+"<br>");


</script>
- - -


호이스팅
=======
- var을 통해 정의된 변수를 유효범위의 최상단으로 올리는 것
- 선언과 할당의 분리 현상
- *var* 의 유효범위 : 함수의 내부
- 변수 선언문 -> 함수 선언문 순서로 호이스팅

1.
```
mul(1,2);
function mul(input1,input2){
  return input1*input2;
}
```

2.
```
mul1(1,2);
var mul1 = function(input1,input2){
  return input1*input2;
}
```

- 위에는 작동하고 아래는 안된다
- 위 두 함수는 다음과 같이 바뀐다.

1.
```
function mul(input1,input2){
  return input1*input2;
}
mul(1,2);
```

2.
```
var mul1;
mul1(1,2);
mul1 = function(input1,input2){
  return input1*input2;
}
```

- 2.는 당연히 에러가 발생한다. 함수의 정의가 되지 않은 상태로 함수를 호출하기 때문

#### 실행결과
<script>
document.write(mul(1,2));
function mul(input1,input2){
  return input1*input2;
}
</script>

<script>
document.write(mul1(1,2));
var mul1 = function(input1,input2){
  return input1*input2;
}
</script>


<!--익명함수
======

추후 추가 예정 -->

- - -

콜백
===

1) 값으로서의 함수
------------

- 함수도 객체 -> 변수에 함수 할당가능(함수 표현식)
- 함수는 값이기 때문에 다른 함수에 인자로 전달 가능
- 반환과 배열값으로 함수 사용 가능
- first class citizen(object): 변수, 매개변수, 리턴 값으로 사용할 수 있는 것들을 의미. 함수가 이에 해당.

2) 콜백
------

- 함수의 인자로 전될되는 함수
- 아래의 예제에서 *sortNumber* 가 callback함수

```
function sortNumber(a,b){
    return b-a;
}
var numbers = [20,1,6,4,3];
//numbers.sort();를 그냥 하면 숫자가 아닌 문자열로 변경하여 정렬하므로 문제가 발생합니다.
numbers.sort(sortNumber);
document.write(numbers);
```
#### 실행결과
<script>
function sortNumber(a,b){
    return b-a;
}
var numbers = [20,1,6,4,3];

numbers.sort(sortNumber);
document.write(numbers);
</script>

3) 비동기 처리
---------

- 비동기 처리 : 특정 코드의 연산이 끝날 때 까지 기다리는 것이 아니라 다음 코드를 수행하는 것 ex) ajax
- 시간이 오래 걸리는 작업이 있을 때, 작업이 완료된 후에 처리해야 할 일을 callback으로 지정하면 해당 작업이 끝난후에 등록한 작업을 실행할 수 있다.

```
function get_data(){
  var data;
  $.get("URL",function(response){
    data = response;
    });
    return data;
}

document.write(get_data());
```
- 위의 실행 결과는 undefined가 된다.
- 비동기 처리에 의해 *return* 을 먼저 시행하는 것이다
- callback을 이용하여 고칠 수 있다

```
var data;
function get_data(callback_func){
  $.get("URL",function(response){
    callback_func(response);
    });
}

get_data(function(data){
  document.write(data);
  });
```

- - -

클로저
=====

- 내부함수가 외부함수의 context(변수)에 접근하는 함수
- 외부 함수는 호출되면 **함수객체** 를 리턴한다.
- 이 함수 객체를 호출하여 사용할 수 있다.

```
function outter(var){
  function inner(){

  }
}
```
- 외부함수가 사라지더라도 내부함수에서 그 context에 접근 가능

```
function outter(){
  var x = 2;
  return function(){alert(x);}
}
var inner = outter();
inner();
```

- 클로저를 이용하면 private variable로 만들 수 있음

```
function outter(pri,pri2){
  return {
    get_val:function(){return pri2+pri;},
    set_second:function(_pri){pri=_pri2;}
  };
}
var x = outter(12,10);
document.write(x.get_val());
x.set_second(11);
document.write(x.get_val());
```
#### Result
<script>
function outter(pri,pri2){
  return {
    get_val:function(){return pri2+pri;},
    set_second:function(pri2_tmp){pri2=pri2_tmp;}
  };
}
var x = outter(12,10);
document.write(x.get_val()+"<br>");
x.set_second(11);
document.write(x.get_val());
</script>

활용
---

```
var arr = []
for(var i = 0; i < 5; i++){
    arr[i] = function(){
        return i;
    }
}
//함수가 호출 될 때 i를 확인함.
for(var index in arr) {
    document.write(arr[index]()+"<br>");
}
```
#### Result
<script>
var arr = []
for(var i = 0; i < 5; i++){
    arr[i] = function(){
        return i;
    }
}
for(var index in arr) {
    document.write(arr[index]()+"<br>");
}
</script>

```
var arr = []
for(var i = 0; i < 5; i++){
    arr[i] = function(id) {
        return function(){
            return id;
        }
    }(i);
}
// 내부 함수에서 외부 함수에 저장된 변수(그 때 저장됨)을 참고하면서 호출
for(var index in arr) {
    document.write(arr[index]()+"<br>");
}
```
<script>
var arr = []
for(var i = 0; i < 5; i++){
    arr[i] = function(id) {
        return function(){
            return id;
        }
    }(i);
}
for(var index in arr) {
    document.write(arr[index]()+"<br>");
}
</script>
- - -

Arguments
==========

- 인자 : 함수에 입력해주는 값
- 매개변수 : 인자를 받는 변수
- 자바스크립트는 인자와 매개변수의 수가 달라도 상관 없다.
- arguments : 자바스크립트의 함수 안에 저장 되어있는 객체. 사용자가 전달한 인자를 받아줌.
- arguments.length vs function_name.length : 전자는 입력된 인자의 개수. 후자는 명시적으로 정의되어 있는 매개변수의 수.

```
function mul(){
  var mul=1;
  document.write(arguments.length+"<br/>");
  for(let i=0;i<arguments.length;++i){
    document.write(i+" : "+arguments[i]+"<br>");
    mul*=arguments[i];
  }
}

mul(1,2,3,4,5);
```
#### Result
<script>
function mul(){
  var mul=1;
  document.write(arguments.length+"<br/>");
  for(let i=0;i<arguments.length;++i){
    document.write(i+" : "+arguments[i]+"<br>");
    mul*=arguments[i];
  }
}

mul(1,2,3,4,5);
</script>

- - -

호출
===

- 함수도 객체이므로 내장된 속성과 메소드가 존재한다.
- *함수이름.apply(context,args)* : args는 배열로 된 인자를 받고. 함수에 주는 인자들을 저장한다. context로 null을 주는 것은 그냥 함수를 호출하는 것과 마찬가지이므로 쓰지 말도록 하자.
- *함수이름.apply(context,args)* 를 시행하면 context에 주어진 객체에 함수를 메소드로 추가하고 그 메소드를 호출한 후에 다시 그 메소드를 삭제하는 것이다.
- 이 방법을 사용하면 각 객체의 속성들에 접근할 수 있다.
- null을 사용하는 것은 전역객체를 context로 하여 실행하는 것이다.

```
obj1={val1:1,val2:2};
function sum(){
  let x = 0;
  for(key in this)x+=this[key];
  return x;
}
document.write(sum.apply(obj1));
```
#### Result
<script>
obj1={val1:1,val2:2};
function sum(){
  let x = 0;
  for(key in this)x+=this[key];
  return x;
}
document.write(sum.apply(obj1));
</script>

- - -

여러 함수들
==========

1) map, filter, reduce
-----------------------

- map : 배열에 규칙을 적용하여 새로운 배열 생성
- filter : 배열의 주어진 조건을 만족하는 것만 선택한 후 배열 생성
- reduce : 배열의 크기를 줄여나가면서 작업 수행

```
//map(callback)
var array = [1,2,3,4];
var newArray = array.map(function(elements){
  return elements*elements;
  });

//filter(callback)
var array = [1,2,3,4];
var newArray = array.filter(function(elements){
  return elements%2==0;
  });

//reduce(callback[,initV])
//initV : 첫번째로 사용되는 인수(default : 0)
//pre : 마지막 콜백에서 반환된 값 or initV
//cur : 배열에서 현재 처리되고 있는 요소
//curI : 배열에서 현재 처리되고 있는 요소의 index
//array : 원 배열
var sum = [1,2,3,4].reduce(function(pre,cur,curI,array){
  return pre+cur;
  });
```

2) setInterval, setTimeout, clearInterval
-----------------------------------------

- setInterval : 일정 시간동안 반복해서 시행
- clearInterval : setInterval 해제
- setTimeout : 지연시간 발생 시킨 후 특정 함수 호출

```
//setTimeout(callback,delayTime)
setTimeout(alert("hi"),1000)//1초후 alert

//setInterval(callback,milliseconds)
var SI = setInterval(alert("hi"),3000);//3초마다 alert

//clearInterval(setInterval변수)
clearInterval(SI);
```

- - -

Arrow Function
==============

- **화살표 함수(Arrow function)** 은 ES6의 문법이다
- [공식 문서](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98)
- **this, arguments, super, new.target** 을 binding하지 않음(사용할 수 없음)
- 생성자로 사용 불가
- 주로, 메소드가 아닌 함수에 사용

```
//기존
function(){

}

//arrow function
()=>{}
```

- 아래는 this와 관련된 내용이다

```
//두가지 경우
function mother(){
  this.value=1;
  setTimeout(function(){
    //this != mother
    console.log(this.value);
  },0);
}

function mother(){
  this.value=1;
  setTimeout(()=>{
    //this == mother
    console.log(this.value);
  },0);
}
```

- 다음과 같이 사용 가능하다(이 함수는 생성자로 쓰일 수 없다)

```
var func = (x,y)=>{return x+y;}
```
- - -

참고자료
------
[변수와 유효범위](https://yuddomack.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EB%B3%80%EC%88%98%EC%99%80-%EC%8A%A4%EC%BD%94%ED%94%84%EC%9C%A0%ED%9A%A8%EB%B2%94%EC%9C%84?category=754152)\\
[filter,map,reduce](https://lee-mandu.tistory.com/9)\\
[Arrow function](https://mygumi.tistory.com/229?category=642142)
