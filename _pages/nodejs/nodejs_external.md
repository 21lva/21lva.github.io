---
layout: post
title:  "외부 모듈"
date:   2019-07-31 02:02:59
author: Inhyuk
category: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_external.md
---

- npm(node packaged manager) : 모듈을 설치, 공유

```console
$ npm install ejs jade
```

외부 모듈
========

1) ejs
--------

- ejs 모듈 : 템플릿 엔진 모듈. 특정 형식의 문자열을 HTML 형식의 문자열로 변환
- render() : ejs 파일을 불러서 HTML 파일로 렌더링 하는 메서드. 두 번째 매개변수로 전달하고자 하는 값을 전달 할 수 있다.

```js
var http = require("http");
var fs = require("fs");
var ejs = require("ejs");

http.createServer(function(request,response){

  //ejs 파일 읽기
  fs.readfile('ejsPg.ejs','utf8',function(error,data){
    response.writeHead(200,{'Content-Type':'text/html'});
    //ejs rendering
    response.end(ejs.render(data),{
      val:"Hello"
      });
    });
  });
```

- ejs 파일은 다음과 같이 작성되어 있다.

```html
<% 자바스크립트 코드 %>
<%= 값 %> : 값을 출력
```

```html
{% raw %}
<% var hi = "hi" %>
<h1><%= hi %></h1>
<h2><%= val %></h2><!-- val 값은 render 메서드 호출 시 전달하였다. -->
<% for(let i=0;i<10;++i){ %>
  <h2>The square of <% =i %> is <% =i*i %></h2>
<% } %>

<% if(true) {%>
  <h1>hi</h1>
<% }else{ %>
  <h2>hello</h2>
<% } %>
{% endraw %}
```

2) jade
--------

- 현재는 pug 라는 모듈을 jade 대신 사용한다.
- ejs 처럼 템플릿 엔진이다
- compile(string,option) : jade 문자열을 HTML 문자열로 바꿀 수 있는 **함수를 생성**

```js
var http = require('http');
var jade = require('jade');
var fs = require('js');

http.createServer(function(request,response){

  //jade 파일 읽기
  fs.readfile('jadePg.jade','utf8',function(error,data){
    var jadeFn = jade.compile(data);

    response.writeHead(200,{'Content-Type':'text/html'});
    response.end(fn({
      val1 : "this is from outside",
      val2 : "this is also from outside"
      }));
    });
  });
```

- jade 파일의 모습
- div태그는 div를 쓰는 것 말고도 단순히 id나 class를 입력하면 된다.
- *- 코드* : 자바스크립트 코드 입력
- *#{value}* : 데이터 출력
- *= value* : 데이터 출력

```jade
html
    head
        title First page
    body
        h1 This is title.
        h2 #{val1} ???
        h2= val2 !!!
        a(href="www.google.com") Go to google
        .class1
            this is a class div
        #id1
            this is a id div
        - for(let i =0 ;i<10;++i){
              p
                  a(href="www.google.com") Go to google
        - }
```

3) supervisor
------------

- 코드의 변화가 생길 시 자동으로 서버를 재실행 시키는 모듈. 전역 모듈이기 때문에 터미널에서 직접 사용

```console
$ sudo npm install -g supervisor
```

- 지속적으로 실행 시키고 싶은 모듈을 supervisor로 실행시킨다.
- 서버 프로그램에만 쓰도록 하자.

```console
$ supervisor main.js
```

4) forever
----------

- node.js는 단일 스레드 기반이이 깨문에 예외가 발생하면 웹 서비스 전체가 죽는다
- forever : 예외 상황에 대처하는 모듈. 전역 모듈이기 때문에 터미널에서 직접 사용

```console
$ npm install -g forever
```

- 기능은 다음과 같다

```console
// 자바스크립트를 forever로 실행. 예외가 발생해도 꺼지지 않는다
$ forever a.js

// 실행되고 있는 웹서버 확인
$ forever list

// 프로세스 종료
$ forever stop 0
```
