---
layout: post
title:  "이벤트"
date:   2019-05-17 01:02:59
author: Inhyuk
categories: Javascript
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_event.md
---


이벤트
======

- 이벤트 : 유저에 의해 어떤 동작이 발생하는 것을 의미
- 이벤트 타입(=이벤트 이름) : 발생한 이벤트의 종류를 나타내는 문자열
- 이벤트 타겟 : 이벤트가 일어날 대상
- 이벤트 객체 : 이벤트와 관련 있는 객체. 이벤트 타입, 이벤트 타겟을 프로퍼티로 가짐. 이벤트 리스너가 호출될 때 인자로 전달됨.
- 이벤트 핸들러(이벤트 리스너) : 이벤트가 발생했을 때 동작하는 코드
- 이벤트 등록 : 이벤트의 대상에 이벤트 핸들러를 지정해주는 것

1) 등록 방식
--------

```html
//인라인 방식
<input type="button" onclick="alert('hello');">

//프로퍼티 리스너 방식
<input type="button" id="event-target" >
<script>
  var t=document.getElementById("eventTarget");
  t.onclick(function(){
    alert("hello");
    })
</script>

//addEventListener() 메서드 사용. 선호되는 방식
<input type="button" id="eventTarget">
<script>
  var t=document.getElementById("event-target");
  t.addEventListener("click",function(){
    alert("hello");
    })
</script>

//하나의 객체에 여러 이벤트 핸들러 등록
<script>
var btn = document.getElementById("btn");       
btn.addEventListener("click", clickBtn);       
btn.addEventListener("mouseover", mouseoverBtn);
btn.addEventListener("mouseout", mouseoutBtn);
function clickBtn() {
    document.getElementById("text").innerHTML = "버튼이 클릭됐어요!";
}
function mouseoverBtn() {
    document.getElementById("text").innerHTML = "버튼 위에 마우스가 있네요!";
}
function mouseoutBtn() {
    document.getElementById("text").innerHTML = "버튼 밖으로 마우스가 나갔어요!";
}

//이벤트 핸들러 삭제
btn.removeEventListener("mouseout", mouseoutBtn);
</script>
```

- 이벤트 호출 순서
1. 대상이 되는 객체나 요소에 프로퍼티로 등록한 이벤트 핸들러 호출
2. addEventListener()로 등록된 이벤트 핸들러 호출

- 이벤트 등록 메서드

|load()|선택한 이미지나 프레임의 연동된 소스의 로딩이 완료된 후 발생|
|ready()|지정한 문서 객체의 로딩이 완료된 후 발생|
|error()|이벤트 대상의 오류가 발생시|
|click()|클릭 시|
|dbclick()|더블 클릭시|
|mouseout()|마우스 포인터가 선택한 요소에서 벗어났을 때|
|mouseover()|선택한 요소에 마우스 포인터를 올렸을 때|
|hover()|선택한 요소에 마우스 포인터를 올렸을 때와 벗어났을 때|
|mousedown()|선택한 요소를 마우스로 눌렀을 때|
|mouseup()|선택한 요소에 마우스를 눌렀다 뗐을 때|
|mousemove()|마우스를 움직였을 때|
|scroll()|스크롤 시|
|focus()|선택한 요소에 포커스가 생성되었을 때 혹은 선택한 요소에 강제로 포커스 생성|
|focusin()|선택한 요소에 포커스가 생성되었을 때|
|focusout()|선택한 요소로부터 포커스가 다른 곳으로 이동시|
|change()| 이벤트 대상의 입력요소의 값이 변경시 혹은 포커스 이동시|
|keypress()| 키보드를 눌렀을 때. 문자키를 제외한 키 코드값 반환|
|keydown()| 키보드를 눌렀을 때. 모든 키 값 반환|
|keyup()|키를 뗐을 때|

- script가 사용하려는 객체보다 위에 있으면 동작을 하지 않는다
- 다음과 같이 **window.onload** 를 사용하여, window가 로드가 된 이후에 등록하도록 한다

```html
<script>
window.onload = ()=>{  
  var t=document.getElementById("event-target");
  t.onclick(()=>{
    alert("hello");
    })
};
</script>
<input type="button" id="event-target" >
```

- - -

이벤트 전파
==========

- 이벤트 전파 : 이벤트 발생 시 이벤트 핸들러를 실행시킬 대상 요소를 결정하는 과정. 버블링과 캡쳐링의 두 가지 방식 존재.
- 버블링 방식 : 이벤트 발생한 요소를 기점으로 DOM트리를 따라서 위로 전파됨(bottom-up). window객체까지 계속 이어짐.
- 캡쳐링 방식 : 이벤트 발생한 요소까지 최상위부터 아래로 전파(top-down). addEventListener()의 세번째 인수로 true를 전달하면 됨.
- 이벤트 호출 -> 타깃까지 내려감(캡쳐링) -> 다시 최상위로 올라옴(버블)
- 위 과정은 예상치 못한 결과를 발생 시킴. ex) 부모와 자식에 모두 클릭이 있을 시 자식을 클릭하면, 자식 이벤트 핸들러 호출 후 부모도 발생(stopPropagation을 통해서 방지)

- 이벤트 전파, 기본 동작 취소 관련 메서드

```js
//preventDefault() : 현재 이벤트의 기본 동작을 취소

//stopPropagation() : 상위로 전파되는 것을 방지(버블링 방지)

//stopImmediatePropagation() : 상위 뿐만 아니라 현재 레벨의 다른 이벤트 동작도 방지.

//return false : jQuery 사용시 preventDefault + stopPropagation, 미사용시 preventDefault의 역할
$("#a").on("click",function(event){
    $("#console").append("<br>A 클릭");
    return false;
});
```

- - -

자바스크립트 이벤트 동작 방식
============================

- 자바스크립트는 **Event driven** 이다
- Javascript Engine : 자바스크립트 코드를 해석하는 인터프리터.
1. Call stack
2. Task Queue(Evnet Queue)
3. Heap
으로 구성


Call Stack
------------

- 자바스크립트는 하나의 call stack만 사용한다
- 함수가 실행되면 해당 함수의 명령을 stack에 쌓고, 하나씩 호출(pop)한다.

Event loop
-----------

- call stack을 주시하다가 call stack이 빌 경우, task queue에서 새로운 작업을 call stack에 쌓는다.

Task Queue
-----------

- 처리해야 하는 task(콜백)들이 call stack에 쌓기전에 거치는 장소
- **FIFO** 로 처리
- **콜백 함수들은 call stack에 쌓이지 않고 task queue에 push된다.**

```js
setTimeout(()=>{
  console.log("This is first");
  },0);
console.log("This is second");
```

- 위의 결과는 *This is second* 가 먼저 된다.
- *setTimeout* , *XMLHttpRequest* 같은 함수는 Javascript engine이 아닌 **Web API** 에 정의되어 있다.
- stack에 쌓인 함수에 비동기 처리가 필요한 부분은 **Web API** 에서 처리되고 결과는 task queue에 전달한다
- 예를 들어, ex. setTimeout(f,0)은 Web API에 의해 0초후에 f가 task queue에 추가된다. 이 방법은 특정 함수 후에 함수를 실행시키기 위해서 사용되기도 한다

- 다음과 같은 경우도 있다

```js
$('.btn').click(function() {
    try {
        $.getJSON('/api/members', function (res) {
        });
    } catch (e) {
        console.log('Error : ' + e.message);
    }
});
```

- *$.getJSON* 은 *XMLHttpRequest* 를 이용한다.
1. 콜백함수는 task queue에 추가되고, *$(".btn").click* 함수는 call stack에서 제거 된다.
2. *$.getJSON* 가 call stack에서 실행되고, 에러를 야기하지만 try catch에 의해 잡히지는 않는다.(이미 밖 함수가 call stack에서 제거 되었기 때문)

- - -

참고 자료
========

- [이벤트 핸들러](http://tcpschool.com/javascript/js_event_eventListenerRegister)
- [이벤트 루프1](https://asfirstalways.tistory.com/362)
- [이벤트 루프2](https://meetup.toast.com/posts/89)
- [이벤트 전파](https://webclub.tistory.com/186)
