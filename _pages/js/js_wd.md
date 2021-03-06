---
layout: post
title:  "javascript 4 - Window & BOM"
date:   2019-05-17 02:02:59
author: Inhyuk
categories: Javascript
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_wd.md
---


객체화
=====

![계층]({{site.baseurl}}/post_img/{{page.name}}/sta.png)

- 웹브라우저는 구성요소들이 객체화 되어 있음. 이 객체들을 이용하여 제어
- 전역객체(window) 밑으로(프로퍼티) DOM, BOM, javascript 객체 등이 존재
- Browser Object Model(BOM) : 웹페이지의 내용을 제외한 브라우저의 각종 요소들을 객체화한 것
- Document Object Mode(DOM) : 웹페이지의 내용을 제어하는 객체.

```html
<html>
<body>
  //BOM
  <input type="button" value='Hello' onclick="alert(window.location)" />
  //DOM
  <input type="button" value='Hello' onclick="document.write('hi')" />
</body>
</html>
```


- 웹브라우저의 창, 프레임등을 제어할 수 있도록 한다

Window
=======

- 모든 객체가 소속된 객체
- 전역객체이면서 창을 뜻한다
- 모든 전역 객체와 전역 함수는 전역객체의 프로퍼티와 메소드이다
- 생략 가능하다
- **ECMAScript** 에서는 *Global* 객체라고 부른다

- - -

사용자와 상호작용
==============

- *alert* : 경고창. 정보 제공 및 디버깅에 사용
- *confirm* : 사용자에게서 확인을 받음. *true/false* 를 리턴
- *prompt* : 사용자로부터 값을 입력 받는 함수.

```html
<html>
  <body>
    <input type="button" value="alert" onclick="alert('hello world');" />
    <input type="button" value="confirm" onclick="func_confirm()" />
    <input type="button" value="prompt" onclick="func_prompt()" />
    <script>
        function func_confirm(){
            if(confirm('ok?')){
                alert('ok');
            } else {
                alert('cancel');
            }
        }
        function func_prompt(){
          // good? 을 출력하고 yes를 입력받으면 welcome, 다른 값을 입력 받으면 fail의 창을 띄움
            if(prompt('good?') === 'yes'){
                alert('welcome');
            } else {
                alert('fail');
            }
          }
    </script>
  </body>
</html>
```

- - -

Location
=========

- 문서의 주소와 관련된 객체.
- 현재의 url을 검색/수정 등을 할 수 있음.

#### 현재의 URL

```js
location.toString();
location.href;//좀 더 선호됨
```

#### URL 파싱
- 의미에 따라 별도의 프로퍼티 제공
- protocol, host, port, pathname, search, hash 등이 존대

```js
location.protocol, location.host, location.port, location.pathname, location.search, location.hash
```

#### 주소 변경
```js
location.href="https://www.google.com"//해당 url로 이동
location="https://www.google.com"// 해당 url로 이동
locaion.reload()//새로 고침
```

- - -

Navigator
=========

- 브라우저의 정보를 제공하는 객체
- 호환성 문제에 주로 사용
- 크로스브라우징 이슈 : 브라우저마다 다르게 동작하는 문제
- 웹표준 : 크로스브라우징 이슈를 해결하기 위하여 만든 규칙

```js
console.dir(navigator) //navigator의 프로퍼티와 메소드들을 보여줌
navigator.appName //웹브라우저 이름
navigator.appVersion //브라우저의 버전
navigator.userAgent //브라우저가 서버에 전송하는 USER-AGENT HTTP 헤더의 내용.
navigator.platform // 브라우저가 작동하는 운영체제에 대한 정보
```

기능 테스트
---------

- 모든 브라우저에서 직접 테스트하는 것은 쉽지 않음
- 기능 테스트+navigator를 통해서 해당 기능을 브라우저가 제공하는지 확인해야 함.

```js
if (!Object.keys) {
  Object.keys = //keys가 없을 시 동작할 기능 정의
}
```

- - -

창 제어
======

- *window* 객체는 브라우저를 직접 다루는 기능을 제공

```js
open("https://www.google.com/"); //새로운 창에 해당 페이지 열기 + 그 창의 window 객체 리턴
open("https://www.google.com/",_self); //현재 창에 해당 페이지 열기
open("https://www.google.com/",_blank); //새로운 창에 해당 페이지 열기
open("https://www.google.com/",'ot'); //새로운 창에 해당 페이지 열기, 한 번 열리고 나면 그 창에 계속 열기
open("https://www.google.com/", '_blank', 'width=200, height=200, resizable=no'); //사이즈, 창크기 조절 가능 등을 조절할 수 있음
```

- 사용자의 상호작용(버튼을 누르는 등)을 하는 경우가 아닌 팝업의 경우 브라우저가 보안/속도 문제 때문에 차단한다.

```html
<html>
<body>
    <script>
    window.open('demo2.html');
    </script>
</body>
</html>
```
