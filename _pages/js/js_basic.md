---
layout: post
title:  "기본 문법"
date:   2019-05-15 01:02:59
author: Inhyuk
categories: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_basic.md
---

- **Javascript** 는 웹에서 프로그래밍을 담당하는 언어이다.
- 웹에서 동적으로 어떤 행동을 하는 언어이다.
- **php, java, python** 가 해오던 웹서버 기능도 담당하게 되었다(node.js)
- 웹 전체에서 자바스크립트의 비중이 늘어나는 중
- 웹 뿐만 아니라 다양한 분야에서 사용 중

- **Atom** 의 package를 다루고 빅스비좀 쓰려고 하니 Javascript가 필요한 것 같아서 이제부터 공부하려 한다.
- 참고한 책 또는 인터넷 서적은
1. [생활코딩 자바스크립트 강의](https://www.opentutorials.org/course/743)
2. [Do it! 자바스크립트 + 제이쿼리 입문](http://www.yes24.com/Product/Goods/59461086)
- 그 외에 중간중간에 찾은 정보는 각 페이지 하단에 참고자료로 적어놓았다.
- C언어 및 다른 언어를 알고 있다는 전제하에 정리하였다(그 만큼 생략이 많다는 뜻)

- - -

HTML과 함께 사용하기
=================

1) inline 방식
----------

- 태그와 연관된 스크립트를 분명히 알 수 있음
- 길어지면 보기 힘듬

```html
<html>
<body>
  <input type="button" onclick="alert('hello');" value='Hello'>
</body>
</html>
```

2) script 방식
-----------

- html과 자바스크립트 코드를 분리하여 보게 되어서 가독성이 좋다

```html
<html>
<body>
  <input type="button" value='Hello' id="but">
  <script type="text/javascript">
    var but = document.getElementById("but");
    but.addEventListener("click",function(){
      alert("hello");
      })
  </script>
</body>
</html>
```

- 아래처럼 외부 파일로 분리할 수도 있다.
- script 파일은 head 태그가 아닌 body 태그 하단에 위치하는 것이 좋다. 혹은 *onload* 를 사용하면 가능

```html
<html>
<body>
  <input type="button" value='Hello' id="but">
  <script type="text/javascript" src="script.js"> </script>
</body>
</html>
```
숫자, 문자, 조건문, 배열, 반복문을 다룬다.\\
내용은 계속 추가된다. 공부할 때 마다 ^^

자료형
=====

1) 숫자
---

- 정수, 실수 등이 존재
- html에서 수식을 적으면 수식 자체가 보이지만, js는 값이 계산됨

```html
<body>
  1+1<br>
  <script type="text/javascript">
    document.write(1+1);
  </script>
</body>
```

- *Math* 를 이용하여 여러 개산 가능(pow, round, ceil, sqrt, random 등)
- *document.write* 는 document의 write 메소드를 불러와서 화면에 출력한다.

```html
<body>
  <script type="text/javascript">
    document.write(Math.pow(2,4)+"<br>");
    document.write(Math.round(2.4)+"<br>");
    document.write(Math.ceil(2.4)+"<br>");
    document.write(Math.sqrt(7)+"<br>");
    document.write(Math.random()+"<br>");
  </script>
</body>
```


2) 문자와 문자열
-----------

- 한 문장을 나타낼 때 ' '나 " "로 감싸준다
- 문자열도 객체이다. 메소드가 존재한다는 뜻
- 당연히 '로 열리면 '로 닫혀야 하고, "로 열리면 "로 닫아야한다
- ' ' 안에 ' '를 쓰고 싶으면 \'를 사용한다. 큰 따옴표도 마찬가지(escape라고 한다)
- 문자열을 결합하고 싶으면 *+* 를 사용한다.
- 문자열과 숫자를 결합하려면 *문자열+(숫자)* 로 해준다.
- *indexOf* 메소드는 문자열의 해당 문자의 index를 알려준다. 해당 문자가 여러 개이면 처음을 알려준다.

```html
<script type="text/javascript">
  document.write("hi"+"hello"+"<br>");
  document.write("hi"+"1+2"+"<br>");
  document.write("hi"+(1+2)+"<br>");
  document.write("hiiiii".indexOf("i"));
</script>
```


- - -

변수
===

- 다른 언어의 변수의 개념과 거의 비슷
- *var* 를 사용하면 동적으로 자료형 결정
- 영문자, _ , $(*jQuery에서 많이 사용*) 로 시작해야 한다
- 자료형에는 Number, String, Boolean, Null, Undefined, 객체가 존재
- 자바스크립트의 변수는 **wrapper객체** 로 만들어진다.
- **var x = 1** 는 1을 가지는 Number객체를 생성하고 그 객체를 x가 참조한다.
- Undefined : 값을 할당하지 않은 변수(정의가 안됨)
- Null : 값이 없음(값이 없다를 부여받음)
- 자바스크립트의 문자열은 immutable. 문자열에 수정을 가하면 새로운 문자열이 생성된다.
- 각 자료형은 *typeof(변수)* 로 확인 가능

```html
<script type="text/javascript">
  var x;
  document.write(typeof(1)+"<br>");
  document.write(typeof(1.2)+"<br>");
  document.write(typeof('hello')+"<br>");
  document.write(typeof(1==1)+"<br>");
  document.write(typeof(1=="1")+"<br>");
  document.write(typeof(x)+"<br>");
  document.write(typeof(1==="1")+"/ value "+(1==='1')+"<br>");
</script>
```


- *1=="1"* 을 *true* 라고 하는 것을 알 수 있다. 이는 자바스크립트의 두 동등비교 연산자 *==(equal operator)* 와 *===(strict equal operator)* 에 존재하는 차이이다. *===* 는 값 뿐만 아니라 타입과 형시도 같은지를 정확히 판단한다.

- - -

주석문
=====

- html에서는 *<!-- -->* 를 사용하고, 자바스크립에서는 *//* 또는 */\* \*/* 를 사용한다.

```html
<script type="text/javascript">
  document.write(Math.pow(2,4)+"<br>");
  //주석1
  /* 주석2
  여러 줄 가능
  */
</script>
```


- - -

비교연산
======


== 연산자
--------

- == 는 값이 같은지를 확인하고 ===는 같은 값과 타입인지를 확인(값, 타입 등)
- null과 undefined는 둘 다 값이 없기 때문에 ==에서는 같다고 나오지만, 값이 없음을 부여받았냐 아니냐의 차이에 의해 ===에서는 다르다고 구별함
- ==는 true와 1을 같다고 생각

```html
<script type="text/javascript">
  var x=1,y=2,z="2";
  document.write((x===y)+"<br>");
  document.write((y==z)+"<br>");
  document.write((y===z)+"<br>");
  document.write((k===z)+"<br>");
  document.write((null==undefined)+"<br>");
  document.write((null===undefined)+"<br>");
  document.write((1===true)+"<br>");
  document.write((1==true)+"<br>");
</script>
```


!= 연산자
--------

- !==는 ===의 연산자의 반대
- !=는 == 연산자의 반대

```html
<script type="text/javascript">
  var x=1,y=2,z="2";
  document.write((x!==y)+"<br>");
  document.write((y!=z)+"<br>");
  document.write((y!==z)+"<br>");
  document.write((k!==z)+"<br>");
  document.write((null!=undefined)+"<br>");
  document.write((null!==undefined)+"<br>");
  document.write((1!==true)+"<br>");
  document.write((1!=true)+"<br>");
</script>
```



<br>[== vs === ](https://dorey.github.io/JavaScript-Equality-Table/)에서 확인

- - -

조건문
=====

- c의 문법과 거의 비슷
- *0, '', undefined, null, NaN* 등은 false로 간주

- - -

반복문
====

- *while, do while, for* 는 c와 비슷하다.
- *break, continue*

forEach
-------

- Array, Map, Set 객체의 메소드
- 각 객체의 요소에 접근하여 동작 수행


for in
-------

- 객체의 속성들에 반복하여 작업 수행. 모든 객체에서 사용 가능
- 객체의 속성들의 key값을 얻어냄

for of
------

- *for in* 과 비슷
- 객체가 [Symbol.iterator] 속성을 가지고 있어야만 사용 가능
- 이 부분은 추가 예정

```html
<script type="text/javascript">
  var arr=[1,2,3];
  arr.forEach(v=>{document.write((v*v)+"<br>")});
  for(const key in arr){
    document.write((v*v)+"<br>");
  }
  for(const key of arr){
    document.write((v*v)+"<br>");
  }
</script>
```

