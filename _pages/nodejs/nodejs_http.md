---
layout: post
title:  " http 모듈"
date:   2019-07-28 02:02:59
author: Inhyuk
categories: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_http.md
---


http 모듈
=========

- 요청 메세지 : 클라이언트가 서버에 보내는 요청
- 응답 메세지 : 서버가 클라이언트에 보내는 응답

1) server
----------

- server 객체 : server를 다루는 객체. EventEmitter를 상속받음
- createServer() : server 객체를 만드는 http모듈의 메서드
- server.listen(port[,callback]) : server 실행
- server.close([callback]) : serve 종료
- port : 네트워크의 연결되는 포트 번허. 0~65535개 존재.
- server의 이벤트 :
  - request : 클라이언트의 요청시 발생하는 이벤트. 별도로 지정하지 않고 createServer 의 callback으로 이벤트 리스너 등록 가능.
  - connection : 클라이언트가 접속할 때
  - close : 서버가 종료될 때
  - checkContinue : 클라이언트가 지속적으로 연결을 하고 있을 때
  - upgrade : 클라이언트가 http 업그레이드를 요청할 때
  - clientError : 클라이언트가 오류가 발생할 때

```js
//http 모듈
var http = require("http");

//server 객체 생성
var server = http.createServer();

//server 실행. port 번호 = 52273
server.listen(52273,function(){
  console.log("Sever Running at 127.0.0.1:52273");
  });

server.on("request",function(){
  console.log("Request occured");
  });


//serve 종료
server.close(function(){
  console.log("Sever is closed");
  });
```

2) response
------------

- 클라이언트에 웹페이지를 제공하려면 응답 메세지가 필요
- response 객체 : 응답 메세지에 관여하는 객체
- writeHead(statusCode[,statusMessage,headers]) : 응답 헤더를 작성
- end([data],[enconding],[callback]) : 응답 본문을 작성
- status code : 웹페이지에서 오류 혹은 페이지를 제공할 수 있도록 상태를 알려주는 코드.
  - 1xx : 처리 중
  - 2xx : 성공
  - 3xx : 리다이렉트(다른 곳으로 연결)
  - 4xx : 클라이언트 오류. 404(page not found)
  - 5xx : 서버 오류


```js
require("http").createServer(function(request,response){
  response.writeHead(200,{"Content-Type":"text/html"});
  response.end("<h1>Hi</h1>");
  }).listen(52773,function(){
    console.log("Server Running at 127.0.0.1:52773");
    });
```

- 파일 시스템을 이용하여 html 페이지를 제공하려면 다음과 같이 한다.

```js
var fs = require("fs");

require("http").createServer(function(request,response){
  fs.readFile("HTMLPage.html",function(error,data){
    response.writeHead(200,{"Content-Type":"text/html"});
    response.end(data);
    });
  }).listen(52773,function(){
    console.log("Server Running at 127.0.0.1:52773");
    });
```

- Content-Type으로 제공하는 값은 여러가지이다.
  - text/plain : 기본 텍스트
  - text/html : html 문서
  - text/css : css 문서
  - text/xml : xml 문서
  - image/jpeg : jpg/jpeg 그림
  - image/png : png 그림
  - video/mpeg : mpeg 비디오
  - audio/mp3 : mp3 음악파일

```js
var fs = require("fs");

require("http").createServer(function(request,response){
  fs.readFile("StarWars.jpeg",function(error,data){
    response.writeHead(200,{"Content-Type":"image/jpeg"});
    response.end(data);
    });
  }).listen(52773,function(){
    console.log("Server Running at 127.0.0.1:52773");
    });
```

#### 쿠키

- 쿠키 : 키와 값이 들어있는 작은 데이터 조각. 서버와 클라이언트에 모두 저장 가능. 일정 기간 동안 저장. response 데이터를 이용하여 클라이언트에 저장.
- 헤더의 Set-Cookie 속성에 등록

```js
require("http").createServer(function(request,response){
  response.writeHead(200,{
    "Content-Type":"text/html",
    "Set-Cookie":[
    "id = 21lva;Expires = "+date.toUTCString(),
    "Value = 12"
    ]
    });
  response.end("<h1>Hi</h1>");
  }).listen(52773,function(){
    console.log("Server Running at 127.0.0.1:52773");
    });
```

- Expires : 쿠키의 만료 시간을 표준시간으로 작성
- maxAge : 쿠키의 만료 시간을 밀리 초로 작성


#### 강제 이동

- 서버에서 특정 페이지로 강제로 이동시킬 수 있다. location 속성을 사용

```js
require("http").createServer(function(request,response){
  response.writeHead(302,{
    "Location" : "www.google.co.kr"
    });
  response.end();
  }).listen(52773,function(){
    console.log("Server Running at 127.0.0.1:52773");
    });
```

3) request
------------

- request 객체 : 사용자가 요청한 내용에 반응하는 객체. server 객체의 request 이벤트 발생 시 리스너에 첫 매개변수로 들어간다.
- 프로퍼티
  - method : 요청 방식
  - url : 요청한 url
  - headers : 요청 메시지 헤더
  - trailers : 요청 메시지 트레일러
  - httpVersion : http 프로토콜 버전

- url을 이용한 page 구분

```js
var http = require("http");
var fs = require("fs");
var url = require("url");

http.createServer((request,response)=>{
  var pathname = url.parse(request.url).pathname;

  if(pathname==="/"){
    fs.readFile("index.html",function(error,data){
      response.writeHead(200,{"Content-Type":"text/html"});  
      response.end(data);
      });
  }
  else if(pathname==="/OtherPage"){
    fs.readFile("other.html",function(error,data){
      response.writeHead(200,{"Content-Type":"text/html"});  
      response.end(data);
      });
  }
  }).listen(52273);
```

- 또는, method 유형에 따라 분류할 수 있다.

```js
var http = require("http");

http.createServer((request,response)=>{
  if(request.method==="GET"){

  }
  else if(request.method==="POST"){

  }
  }).listen(52273);
```

- GET과 POST의 매개변수를 추출하는 방법이다(POST 방식은 request 이벤트 발생 후 data 이벤트로 값을 전달한다)

```js
var http = require("http");
var url = require("url");

// GET
http.createServer((request,response)=>{
  var query = url.parse(request.url,true).query;
  var id = query.id;
  var password = query.password;

  response.writeHead(200,{"Content-Type":"text/html"});  
  response.end(JSON.stringify(query));
  }
}).listen(52273);

//POST
http.createServer((request,response)=>{
  request.on("data",function(data){
    response.writeHead(200,{"Content-Type":"text/html"});  
    response.end(data);
    });

  }
}).listen(52273);
```

- 클라이언트이 cookie를 추출할 수도 있다.

```js
http.createServer((request,response)=>{
  if(request.headers.cookie){//쿠키가 있는지 확인
    var cookie = request.headers.cookie.split(":").map(function(element){
      var key_val = element.split("=");
      return {key:key_val[0],value:key_val[1]};
      });
  }
  response.end(JSON.stringify(cookie));
}).listen(52273);
```
