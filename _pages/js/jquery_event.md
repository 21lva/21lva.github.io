---
layout: post
title:  "jQuery 2 - 이벤트 & 애니메이션"
date:   2019-06-12 02:02:59
author: Inhyuk
categories: Javascript
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: jquery_event.md
---

- 이벤트 : 사이트에서 유저가 하는 모든 행동
- 이벤트 핸들러(리스너) : 이벤트 발생시(유저가 동작을 했을 시) 실행되는 코드
- [자바스크립트 이벤트 관련 내용]({{site.baseurl}}/javascript/2019/05/17/js5.html)

- - -

이벤트 핸들러 등록
==========

- 이벤트 등록 메서드 : 이벤트 발생시 저장된 코드를 실행시킬 수 있도록 함
- 단독 이벤트 등록 메서드 : 한 동작에 대한 이벤트를 등록
- 그룹 이벤트 등록 메서드 : 2개 이상의 이벤트를 등록

```js
// 단독 이벤트 등록
$("이벤트 대상 선택").이벤트 등록 메서드(function(){자바스크립트 코드});
<button id="but1">Push</button>
$(".but1").click(function(){
  alert("hi");
  });

// 그룹 이벤트 등록
$("이벤트 대상 선택").on("이벤트1 이벤트2 이벤트3.....",function(){자바스크립트 코드});
$("이벤트 대상 선택").on({"이벤트1 이벤트2 이벤트3...":function(){
  자바스크립트 코드
  }})
$("이벤트 대상 선택").on({
  "이벤트1":function(){자바스크립트 코드1},
  "이벤트2":function(){자바스크립트 코드2},
  "이벤트3":function(){자바스크립트 코드3},
  })

//one() : 이벤트를 1개 이상 등록 하고 1회 발생 후 삭제한다.
$("이벤트 대상 선택").one("이벤트",function(){자바스크립트 코드});

$(".but1").on({"mouseover focus":function(){
  console.log("hi");
  }));//2번 방식
$(".but1").on({
  "mouseover":function(){
  console.log("hi");
  },
  "focus":function(){
  console.log("hello");
  }
  );//3번 방식
```

다양한 이벤트 메서드
-----------------------

- 이벤트 강제로 발생 : 사용자에 의해 이벤트가 발생한 것이 아니라 핸들러에 의해 자동으로 이벤트가 발생

```js
// 이벤트 강제로 발생(사용자에 의해 이벤트가 발생한 것이 아니라 핸들러에 의해 자동으로 이벤트가 발생)
$("이벤트 대상").trigger("이벤트 종류");

//이벤트 제거
$("이벤트 대상").off("이벤트 종류");
$(".but1").off("mousover");
```

- - -

Effect & animation
=======

1) Effect
-------
- 선택한 요소를 숨겼다가 보이게 해주는 기능

```js
//기본형
$("요소 선택").효과메서드(소요시간, 가속도, 콜백함수)
```

- 소요시간 : 요소를 숨기거나 노출할 때의 소요되는 시간. slow, normal, fast, 숫자(1000=1초) 등을 지정
- 가속도 : 숨기거나 노출하는 동안의 가속도. swing(시작과 끝은 느리게), linear(일정한속도)등을 지정
- 콜백 함수 : 노출과 숨김 효과가 끝난 후에 실행할 함수(생략 가능)

- hide() : 요소를 숨긴다
- fadeOut() : 요소가 점점 투명해지면서 사라짐
- slideUp() : 요소가 위로 접히며 숨겨짐
- show() : 숨겨진 요소가 노출됨
- fadeIn() : 숨겨진 요소가 점점 선명해짐
- slideDown() : 숨겨진 요소가 아래로 펼쳐짐
- toggle() : hide() <-> show()
- fadeToggle() : fadeOut() <-> fadeIn()
- slideToggle() : slideUp() <-> slideDown()
- fadeTo() : 투명도 적용

```js
$("id1").slideUp(1000,"swing",function(){
  alert("hi");
  });
$("id2").fadeTo(1000,1,function(){
  alert("hello");
  })
//fadeTo() 메서드는 가속도 대신 0~1의 투명도를 지정한다
```


2) 에니메이션

- 요소를 움직이게 하는 기능
- 에니메이션 메서드를 복수로 적용하면 순서대로 실행됨(Queue에 저장된다)

```js
//animtate() : 선택한 요소에 동작효과를 적용하는 메서드
$("요소선택").animate({스타일 속성},적용시간, 가속도, 콜백함수);
$("id1").animate({
  marginLeft:500px,
  },1000,"swing",function(){
    alert("hi"):
    }); //id1을 1초 동안 처음과 마지막에는 느리게 왼쪽으로 500px 이동한 후 alert("hi");를 시행

//stop() , delay() : 정지 or 지연
$("요소선택").stop();
$("요소선택").stop(clearQueue,finish);//clearQueue : true|false , finish : true|false(진행 중인 에니메이션 강제 종료 여부)
$("요소선택").delay(지연시간).후에 적용된 에니메이션();

//queue() : 큐에 함수 추가 or 대기 중인 함수를 배열에 담아 반환. 이후의 에니메이션은 모두 취소
$("요소선택").queue();
$("요소선택").queue(function(){js코드});

//dequeu() : queue()와 달리 대기하고 있는 에니메이션을 제거하지 않음.
$("요소선택").queue(function(){
  //어떤 에니메이션
  $("this").deque();//이거 없으면 queue()이후의 에니메이션은 적용되지 않는다.
}.animate({marginLeft:200px},2000);

//clearQueue() : 진행 중인 에니메이션을 제외한 큐에 담겨진 모든 에니메이션 삭제
```
