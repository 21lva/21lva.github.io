---
layout: post
title:  "DOM"
date:   2019-06-10 02:02:59
author: Inhyuk
categories: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_dom.md
---


- **DOM** 은 **Document object model** 의 약자이다. 윈도우에 로드된 문서를 의미
- **window** 객체의 **document** 프로퍼티를 통해서 사용 가능

- - -

DOM 객체의 구조
=============

- DOM 객체의 구조에는 3가지가 존재한다.

객체 생성과 관련된 상속 관계
-----------------------

![node 계층도]({{site.baseurl}}/post_img/{{page.name}}/node.png)

- Node 객체를 최상위로 시작되는 상속관계 계층
- 객체의 성질과 특징과 관련된 부분


프로퍼티와 관련된 관계
------------------


![Window 계층도]({{site.baseurl}}/post_img/{{page.name}}/window.png)

- 어느 객체의 맴버 또는 프로퍼티로 존재하는가와 관련된 계층도이다.
- 전역객체(window)로 시작하여 내려온다.
- 사진의 위 2층은 **BOM**, 그 외는 **DOM** 객체들이다.

HTML에서의 관계
-------------


![HTML 계층도]({{site.baseurl}}/post_img/{{page.name}}/html.jpeg)
- 문서에 표시되는 관계이다.
- html내에서 태그들의 관계라고 보면 된다.


- - -

각종 객체
========


1) Node 객체
-----------

- DOM의 최상위 객체
- 모든 DOM객체는 Node를 상속받는다.
- 객체끼리의 관계, 객체가 속하는 카테고리, 값을 관리하고, 자식을 추가/수정하는 기능이 존재
- 빈 공간도 Node객체로 사용한다(줄바꿈 같은 경우도 자식 Node로 생각한다)
- 노드 객체의 타입들(nodeType으로 확인가능)
1. Node.ELEMENT_NODE (1)
2. Node.ATTRIBUTE_NODE (2)
3. Node.TEXT_NODE (3)
4. Node.CDATA_SECTION_NODE (4)
5. Node.ENTITY_REFERENCE_NODE (5)
6. Node.ENTITY_NODE (6)
7. Node.PROCESSING_INSTRUCTION_NODE (7)
8. Node.COMMENT_NODE (8)
9. Node.DOCUMENT_NODE (9)
10. Node.DOCUMENT_TYPE_NODE (10)
11. Node.DOCUMENT_FRAGMENT_NODE (11)
12. Node.NOTATION_NODE (12)

- 프로퍼티와 메서드
1. nodeType : 노드의 타입을 의미. 각 타입마다 고유의 숫자가 존재(이름으로도 매칭 가능).
2. nodeName : 노드의 태그 이름
3. childNodes: NodeList(유사배열)가 저장됨. 계속해서 바뀔 수 있음 /ParentNodes, firstChild/ lastChild
5. hasChildNodes() : 자식이 있으면 true / ownerDocument : 문서의 document 객체에 접근.
4. appendChild() / removeChild()

```html
<ol id='ol1'>
<li>First</li>
<li>Second</li>
<li>Third</li>
</ol>
<script>
var x = document.getElementById('ol1');
document.write(x.firstChild.nodeName+"<br/>");//firstChild는 줄바꿈 문자 의미
document.write(x.firstChild.nextSibling+"<br/>");
</script>
```



2) Document 객체
-----------

- DOM의 시작점이자 문서 전체를 의미하는 객체
- **Document** 객체 대신 **HTMLDocument** 객체를 사용.
- **document** 객체는 전역객체의 프로퍼티
- 프로퍼티와 메서드
1. 문서 정보 : document.title : title 요소의 텍스트가 들어가 있음 / document.URL / document.domain
2. 요소 찾기 : getElementById() / getElementsByTagName() / getElementsByClassName()
3. 문서 조작 : write() / writeln() / open() / close()


- tag 안의 tag를 찾고 싶으면 중첩해서 쓰면 된다.


```html
<ol>
<li class="exec">hi</li>
<li class="exec">hello</li>
<li class="exec">bye</li>
</ol>
<script>
var lis = document.getElementsByClassName('exec');
for(let i =0;i<lis.length;++i){
  document.write(lis[i].innerText+"<br>");
}
</script>
```



- **querySelector** : 선택자에 해당하는 객체를 반환. 하나만을 반환, 모든 element를 원하면 **querySelectorAll** 을 사용
- 위 방법은 name, id 등을 제한하지 않고 css선택자를 사용하여 요소를 찾는다.
- **jQuery** 를 사용하고자 하면 html로 로드해야 함.(다운로드 하는 방법도 있음)
- CDN : 온라인 상의 대용량 컨텐츠를 빠르게 전송하는 기술. jquery를 특정 서버에서 전송 받아 사용하도록 한다.
- [jQuery CDN](https://code.jquery.com/jquery/)

```html
<script src="https://code.jquery.com/jquery-3.4.1.js"
			  integrity="sha256-WpOohJOqMqqyKL9FccASB9O0KwACQJpFTUBLTYOVvVU="
			  crossorigin="anonymous"></script>
```

- jQuery 함수는 *$* 를 이용한다.
- **$(css선택자)** 는 jQuery 객체를 리턴

```js
$('li').css("color","red");
```

- 위의 작업은 DOM을 이용하면 다음과 같다.

```js
var lis = document.getElementsByTagName('li');
for(var i =0;i<lis.length;++i){
  lis[i].style.color='red';
}
```

- 태그, 클래스, id 값으로 이용할 수도 있다.

```js
$('tag_name').css("color","red");
$('.class_name').css("color","red");
$('#id_name').css("color","red");
```

- jQuery작업이 끝나면 jQuery 객체를 리턴하기 때문에 연속해서 작업을 하고 싶으면 연속해서 메소드를 호출하면 된다.(체이닝이라고 부른다)

```js
$('.class_name').css("color","red").css('textDecoration','underline');
```

- 다수의 element를 가지는 jQuery객체에는 *[인덱스]* 를 통해서 접근 가능하다.
- map을 통해서 모든 element에 작업을 할 수 있다.
- 얻어낸 jQuery객체의 element들은 DOM element이다.

```html
<ul>
<li class="e1">Number 1</li>
<li class="e1">Number 2</li>
<li class="e1">Number 3</li>
</ul>
<script>
var li = $(".e1");
li.map(function(index,element){
  document.write(index+" "+element+"<br/>");
  $(element).css("color","red");
  })
</script>
```




3) Text 객체
-----------

- Node를 상속 받는 객체
- 태그 사이에 존재하는 글은 Text 객체이다. 태그는 Element 객체이다. ex) <p> 텍스트 객체 </p>
- 공백들도(한줄 띄기 같은 것들) Text 객체이다.(childNodes의 배열을 차지한다.)
- 프로퍼티와 메서드
1. 값 : nodeValue / data
2. 조작 : appendData(String) / deleteData(start,end) / insertData(start,String) / replaceData(start,end,String)

```html
<dir id='p1'>Text Object1</dir>
<dir id='p2'>
<dir>Text Object2</dir>
</dir>
<script>
var pt1 = document.getElementById('p1');
var pt2 = document.getElementById('p2');
document.write((pt1.nodeType===Node.ELEMENT_NODE)+"<br/>");
document.write(pt1.childNodes[0].nodeType===Node.TEXT_NODE);
document.write('<br/>'+pt2.childNodes.length);
</script>
```



4) Element 객체
-----------

- HTMLElement : 모든 HTML(X)Element의 부모, style 프로퍼티를 가진다.
- 요소들을 추상화한 객체이다.
- Element : HTMLElment의 부모. Html을 포함한 다양한 mark up 언어에 사용된다.
- *Element.tagName* : tag의 명칭을 표시, 읽기 전용
- *Element.id* : id 표시
- *Element.className* : class를 표시.
- *Element.classList* : class가 여러 개일 경우 사용.(띄어쓰기를 기준으로 여러 개로 구별)

- 프로퍼티와 메서드
1. element의 정보 : id, title, lang, dir, className
2. 속성 관련 : getAttribute(), setAttribute(), removeAttribute()

- element는 DOM 중 유일하게 attribute 프로퍼티를 가진다.


- - -

HTML element와 collection
========================

- *getElementByID* 와 같은 메소드를 사용하면 **HTMLElement** 객체를 얻는다.
- 하나의 element의 경우 : *HTML(X)Element*
- 복수의 element의 경우 : *HTMLCollection*
- **jQuery** 와 같은 라이브러리 사용 시에는 이 내용들을 라이브러리가 알아서 처리해준다.
- 각각의 객체의 형태에 따라 프로퍼티, 메소드에 차이가 존재한다.
- HTMLLIElement 와 HTMLAnchorElement 등은 모두 HTMLElement의 자식이다(상속되었다)
- [DOM 구조](https://web.stanford.edu/class/cs98si/slides/the-document-object-model.html)
- HTMLCollection 객체는 배열과 비슷하다. 변화에 따라 실시간으로 그 형태와 값이 변한다.


```html
<a id ="anchor" href="google.com">Google</a>
<ol>
<li id="lis">hi<li>
</ol>
<br/>
<input type="button" id="button" value="Button"/>
<br/>
<script>
var t1= document.getElementById("lis");
var t2= document.getElementById("anchor");
var t3= document.getElementById("button");

document.write(t1.constructor.name+"<br/>");
document.write(t2.constructor.name+"<br/>");
document.write(t3.constructor.name+"<br/>");
</script>
```




- - -

참고 자료
========

- [계층도 관련 자료](https://debugjung.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-DOM-%EA%B5%AC%EC%A1%B0)
- [계층 자료](https://feel5ny.github.io/2017/12/26/JS_08_1/)
