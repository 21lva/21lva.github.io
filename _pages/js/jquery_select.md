---
layout: post
title:  "jQuery - 선택 & 객체 & 배열"
date:   2019-06-10 02:02:59
author: Inhyuk
categories: Javascript
tags: js
cover:  "/assets/instacode.png"
name: jquery_select.md
---

**jQuery** 는
- 기본적인 자바스크립트 DOM과 event object에 존재하는 호환성 문제를 해결해 준다.
- 에니매이션 기능을 구현하기 쉽게 해준다.

이 라이브러리를 연동하는 방식은 두 가지 이다.

1. 다운로드 방식 : jQuery 라이브러리를 직접 받아 HTML파일에서 불러오는 방식
2. 네트워크 전송 방식 : 온라인에서 제공하는 제이쿼리 라이브러리 파일을 네트워크를 통해서 HTML에서 불러오는 방식

다운로드 방식은 네트워크에 상관이 없다는 장점이 있지만, 파일을 다운로드 해야하고 import하는 단점이 있다.

[jQuery](https://jquery.com/download/)에 접속하여 다운로드 혹은 네트워크 방식을 사용하자.


- - -

선택자
======

- 선택자 : HTML의 요소를 선택하여 가져온다.
- css 선택자와 비슷하게 디자인 속성을 적용할 수 있으며, css와 달리 동적으로 스타일을 적용할 수 있다.

```js
//요소에 스타일링 적용
$(css선택자).css(속성명, 값);

//요소에 속성을 적용
$(css선택자).attr(속성명,값);

//체이닝 : 메서드를 연속해서 적용하는 기법
$(css선택자).css(속성명, 값).css(속성명, 값).css(속성명, 값).css(속성명, 값);
```

1) 직접 선택자들
------------

```js
//전체 선택자
$("*");

//아이디 선택자
$("#id 이름");

//클래스 선택자
$(".클래스 명");

//요소 선택자
$("태그 이름");

//그룹 선택자
$("선택자1, 선택자2, ....");

//종속 선택자(요소의 id또는 class가 일치할 시 선택. 하위 선택자와 달리 공백이 없음)
$(요소명#id);
$(요소명.클래스명);
```

2) 인접 관계 선택자
-------------------

```js
//부모 요소 선택자
$(요소 선택자).parent();

//하위 요소 선택자(자식 선택자가 아니다)
$(상위요소 하위요소)
ex) $(#id h1);

//자식 요소 선택자
$(부모요소 > 자식요소);
$(부모요소).children(자식요소);
$(부모요소).children() : 모든 자식 요소

//형제 요소 선택자
$(요소선택자).prev();
$(요소선택자).next();
$(요소선택자).prevAll();
$(요소선택자).nextAll();
$(요소선택자).siblings();

//범위 제한 형제 요소 선택자
$(요소 선택자).prevUntil("범위 제한 요소 선택자");
$(요소 선택자).nextUntil("제한 요소 선택자");

//상위 요소 선택자(두 번째는 선택한 요소만을 선택)
$(요소 선택자).parents();
$(요소 선택자).parents("선택 요소");
$(요소 선택자).closet("요소 선택"); // 가장 가까운 상위 요소를 선택
```

- - -

탐색 선택자
==========

- 기본 선택자로 선택한 요소는 배열에 담긴다.
- 기본 선택자로 선택한 요소 중 원하는 요소를 한번 더 탐색한다.

1) 위치 탐색 선택자
------------------

```js
// 처음, 마지막 요소 선택
$("요소:first") || $("요소").first()
$("요소:last") || $("요소").last()

//짝수, 홀수 선택
$("요소:odd");
$("요소:even");

//인덱스 기반 요소 탐색(eq:같음, lt:보다 작음, gt:보다 큼)
$("요소:eq(index)") || $("요소").eq(indx)
$("요소:lt(index)")
$("요소:gt(index)")
$("요소 선택").slice(시작 index, 끝 index)
$("요소선택:nth-child(index)")

//무리 중 선택
$("요소 선택:first-of-type")
$("요소 선택:last-of-type")
```

- nth-child vs eq
1. nth-child(n) : 요소선택에 있는 요소가 아닌 자식들도 전부 카운트 한 후에 n(1부터 카운트)번째 요소를 선택
2. eq(n) : 요소선택에 있는 요소 중 n-1번째(0부터 카운트)요소를 선택한다.


2) 속성 탐색 선택자
--------------

```js
//요소 선택
$("요소[속성]"); //속성이 포함된 요소를 선택
$("요소[속성=값]");//속성의 값을 확인하여 선택
$("요소[속성^=값]");//속성이 값으로 시작되는 경우에 선택
$("요소[속성$=값]");//속성이 값으로 끝나는 경우에 선택
$("요소:hidden");//요소 중 숨겨진 요소만 선택
$("요소:visible");//요소 중 보이는 요소만 선택
$(":selected");//폼이나 select의 선택된 요소를 선택
$(":checked");//radio, check box등에서 체크된 요소만 선택
```

3) 콘텐츠 탐색 선택자
-------------------

```js
//텍스트를 포함하는 요소 선택
$("요소선택:contains(텍스트)")

//하위 요소 중 가장 가까운 하위 요소 선택
$("요소선택").contents()

//해당 요소 중 요소를 포함하는 것만 선택
$("요소선택").has("요소")

//요소의 하위 요소중 해당하는 것만 선택
$("요소").find("요소")
```

- - -

배열
====

```js
$(요소).each(function)

//get([index]) : 선택한 요소들을 배열로 가져옴(index 적을 시 index만 가져옴)

//역순으로 배열에 순차 적용
$("요소선택자").get().reverse().each()

//map : 배열에 저장된 데이터 수 만큼 메서드 실행(오름차순). 결과는 새 배열에 저장하고 그 배열을 반환
//grep : map + 인덱스 오름차순으로 배열의 데이터를 불러옴. 메서드의 반환값이 true인 경우에만 배열에 저장.
$.map(배열,function(value,index){})
$.grep(배열,function(index,value){})

//filter : 배열에 filter 적. grep과 비슷하지만, html 요소에서 바로 사용
$("a").filter(function(index,selector){})

//inArray : 지정된 객체의 인덱스 반환
//isArray : 배열객체인지 확인
//merge : 두 배열을 하나의 객체로 묶는다.
$.inArray(객체, 배열, 시작 index)
$.isArray(object)
$.merge(배열1,배열2)  
```

- grep vs filter(map vs each 도 비슷)
1. grep : 자바스크립트 배열에 사용하는 함수. 자바스크립트의 filter와 비슷하다.(**$.grep(자바스크립트배열,...)**)
2. filter : html 태그를 불러와서 바로 사용하는 함수이다(**$("p").filter(...)**)

- - -

객체 조작
========

- 객체를 조작하는 방법에는
1. 생성
2. 복제
3. 삭제
4. 속성 변경
등이 있다.

1) 속성 메서드
------

```js
//html() : 선택한 요소의 하위 요소를 가져와 문자열로 변환 or 전부 제거 후 변경
$("요소").html();
$("요소").html("새 요소");

//text() : 선택한 요소에 포함되어 있는 전체 텍스트를 가져오기 or 전부 제거 후 변경
$("요소").text();
$("요소").text("새 텍스트");

//attr() : 선택한 요소의  속성값을 가져온다 or 속성값을 변경
$("요소").attr("속성");
$("요소").attr("속성","새 값");
$("요소").attr("속성1":"새 값1","속성2","새 값2");

//removeAttr() : 속성 제거
$("요소").removeAttr("속성");

//addClass() / removeClass() / toggleClass() / hasClass() : 선택한 요소에 클래스 생성/ 제거/ 없으면 생성or있으면 제거/ 있는지 확인
$("요소").addClass("클래스 값")
$("요소").removeClass("클래스 값")
$("요소").toggleClass("클래스 값")
$("요소").hasClass("클래스 값")

//val() : 요소의 value값을 가져오거나 변경
$("요소").val();
$("요소").val("새 값");

//prop() : 선택한 폼 요소의 상태 속성값을 가져오거나 새롭게 변경.
$("요소").prop("checked|selected");
```

2) 수치 조작 메서드
------------------

- 요소의 속성을 조작할 때 사용

```js
//height(),width() : contents의 높이, 너비 변경
$("요소").height();
$("요소").height(100);
$("요소").width();
$("요소").width(100);

//innerHeight(), innerWidth() : padding을 포함한 높이, 너비 변경
$("요소").innerHeight();
$("요소").innerheight(100);
$("요소").innerWidth();
$("요소").innerWidth(100);


//outerHeight(), outerWidth() : margin을 포함한 높이, 너비 변경
$("요소").outerHeight();
$("요소").outerheight(100);
$("요소").outerWidth();
$("요소").outerWidth(100);

//position():위치값 변경.
$("요소").position().[left|right|top|bottom]=값;

//offset(): 문서를 기준으로 위치값 변경.
$("요소").offset().[left|top];

//scrollTop() , scrollLeft() : 스크롤 이동된 값을 변경
$("요소").scrollTop();
$("요소").scrollTop(새 값);
$("요소").scrollLeft();
$("요소").scrollLeft(새 값);
```

3) 객체 편집 메서드
------------------

- 요소를 복제, 생성하는 메서드

```js
//before() /insertBefore(): 선택한 요소의 위(html기준)에 요소 생성
$("요소").before("새 요소");
$("새 요소").insertBefore("요소");

//after() / insertAfter(): 선택한 요소의 위(html기준)에 요소 생성
$("요소").after("새 요소");
$("새 요소").insertAfter("요소");

//append() / appendTo() : 선택한 요소내의 마지막 위치에 새 요소 생성 및 추가
$("요소").append("새 요소");
$("새 요소").appendTo("요소");

//prepend() / prependTo() : 선택한 요소내의 마지막 위치에 새 요소 생성 및 추가
$("요소").prepend("새 요소");
$("새 요소").prependTo("요소");

//clone() : 요소 복제. true이면 event까지 복제. false이면 요소만 복제(기본값)
//empty() : 요소의 모든 하위 요소 삭제
//remove() : 요소 삭제
$("요소 선택").clone([true|false]);
$("요소 선택").empty();
$("요소 선택").remove();

//replaceAll() , replaceWith() : 선택한 요소를 새 요소로 변경
$("새 요소").replaceAll("요소 선택");
$("요소 선택").replaceWith("새 요소");

//unwrap() : 부모 요소 삭제
//wrap() : 새 요소로 감싸기
//wrapAll() : 선택한 요소 한꺼번에 감싸기
//wrapInner() : 선택한 요소의 하위 요소를 새 요소로 감싸기
$("요소 선택").unwrap();
$("요소 선택").wrap("새 요소");
$("요소 선택").wrapAll("새 요소");
$("요소 선택").wrapInner("새 요소");
```

- - -

참고 자료
========
- [eq vs nth-child](http://injulkarnilesh.blogspot.com/2012/10/jquery-difference-between-eq-and-nth.html)
- [grep vs map](https://frontdev.tistory.com/entry/jQuery-grep-%EA%B3%BC-map)
