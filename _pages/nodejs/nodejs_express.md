---
layout: post
title:  "ExpressJS"
date:   2019-07-31 02:02:59
author: Inhyuk
categories: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_express.md
---


ExpressJS
===========

- ExpressJS : http모듈처럼 서버를 다루지만, 여러 가지 기능이 추가된 프레임워크
- nodeJs에 간편하게 웹서버를 만들 수 있는 프레임워크가 여러가지 있다(Express, Koa, Hapi)

```
//Dependencies에 추가
$ npm install express --save

//Dependencies에 추가하지 않음
$ npm install express --no-save
```

1) 서버 생성
----------

- require('express')() : 어플리케이션 객체 생성

```js
var express = require('express');

//서버 생성
var app = express();

//request 이벤트 리스너 설정
app.use(function(request,response){
  response.writeHead(200,{'Content-Type':'text/html'});
  response.end('<h1>hi</h1>');
  });

//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});

```

- express로 서버를 생성하면 request, response에 기능이 추가
- reponse객체에 추가되는 기능 :
  - response.send([body]) : 매개변수의 자료형에 따라 적절한 형태로 응답
  - response.json([body]) : JSON 형태로 응답
  - response.jsonp([body]) : JSONP 형태로 응답
  - response.redirect([status],path) : 웹페이지 강제 이동
  - response.sendStatus(statusCode) : 상태 코드만 보낸다.

- 위의 코드를 send를 이용하여 작성한 것

```js
var express = require('express');

//서버 생성
var app = express();

//request 이벤트 리스너 설정
app.use(function(request,response){

  response.status(200).send("<h1>HI</h1>");
  });

//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});
```

2) 기본 요청
------------

- request 객체에 추가되는 기능
  - params : 라우팅 매개변수 추출
  - query : 요청 매개변수 추출
  - headers : 요청 헤더를 추출
  - header() : 요청 헤더의 속성을 지정 혹은 추출
  - accepts(type) : 요청 헤더의 Accept 속성을 확인
  - is(type) : 요청 헤더의 Content-Type 속성을 확인

```js
var express = require('express');

//서버 생성
var app = express();

//request 이벤트 리스너 설정
app.use(function(request,response){
  var user = request.header("User");
  console.log("The use is %s",user);
  console.log("The headers are %s",request.headers);

  //요청 매개변수 추출
  console.log(request.query.name);
  console.log(request.query.id);
  response.sendStatus(200);
  });

//server 시작
app.listen(52273,function(){
  console.log('Server Running')';
});
```

3) 미들웨어
----------

- use() : express와 http 모듈의 큰 차이점. 여러번 사용 가능. 매개변수로 next를 추가하여 다음에 위치하는 함수를 이용하게 한다.
- next() : 다음 함수에 제어를 전달하는 함수.
- 미들웨어를 이용하면 이전 미들웨어에서 request나 response에 추가한 속성과 메서드를 이용할 수 있다


```js
var express = require('express');

//서버 생성
var app = express();

//첫 미들웨어
app.use(function(request,response,next){
  //이 함수가 미들웨어다   
  console.log('First Middle ware');
  next();
  });

//두번째 미들웨어
app.use(function(request,response,next){
  console.log('Second Middle ware');
  next();
});

//마지막 미들웨어
app.use(function(request,response){
  console.log('last Middle ware');
  response.sendStatus(200);
  });

//server 시작
app.listen(52273,function(){
  console.log('Server Running')';
});
```

4) router 미들웨어
------------------

- express 모듈에 내장되어 있어 미들웨어라고 잘 알지 못한다.
- 페이징 라우팅을 해주는 미들 웨어
- 페이징 라우팅 : 클라이언트에 적절한 페이지를 제공하는 기술
- app를 이용한 메서드
  - get(path,callback[,callback]) : GET 요청 발생 시의 이벤트 리스너 등록
  - post(path,callback[,callback]) : POST 요청 발생 시의 이벤트 리스너 등록
  - put(path,callback[,callback]) : PUT 요청 발생 시의 이벤트 리스너 등록
  - delete(path,callback[,callback]) : DELETE 요청 발생 시의 이벤트 리스너 등록
  - all(path,callback[,callback]) : 모든 요청에 대한 이벤트 리스너 등록

```js
var express = require('express');

//서버 생성
var app = express();

app.get('/a',function(request,response){
  response.send("<h1>hi</h1>");
  });

app.get('/a/:id',function(request,response){
  response.send("<h1>hello"+request.params.id+"</h1>");
  });

//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});
```

- params :  "/:id" 처럼 ":"를 이용한 라우팅 매개변수
- query : ?name=A 와 같은 요청 매개변수

5) static 미들웨어
-----------------

- express 모듈 자체에 내장되어 있는 미들웨어
- 폴더의 위치를 변환할 때 사용

```js
var express = require('express');

//서버 생성
var app = express();

//static 메서드 사용. __dirname은 전역변수이다. 지정한 폴더에 있는 내용을 모두 웹 서버 루트 폴더에 올림
app.use(express.static(__dirname+"/public"));

//파일 제공
app.use(function(request,response){
  response.writeHead(200,{'Content-Type':'text/html'});
  response.end("<img src='/im.jpg' widhth=50% />");
  })

//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});
```

6) morgan 미들웨어
-----------------

- 웹에 요청이 들어왔을 때에 로그를 출력하는 미들웨어
- 외부 모듈이므로 따로 설치해야함

```
$ npm install morgan
```

```js
var express =require("express");
var morgan = requir('morgan');

var app = express();

app.use(morgan('combined'));
app.use(function(request,reponse){
  response.send("<h1>Express</h1>");
  });

//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});
```

- 토큰들을 조합하여 원하는 로그를 출력할 수 있다.
  - :req[header] : 요청 헤더
  - :res[header] : 응답 헤더
  - :http-version : HTTP 버전
  - :response-time : 응답 시간
  - :remote-addr : 원격 주소
  - :date[format] : 요청 시각
  - :method : 요청 방식
  - :url : 요청 url
  - :referrer : 이전 url
  - :User-Agent : 사용자 에이전트
  - :status : 상태 코드

- 기본 형식 : 사용자가 편리하게 사용하도록 토큰들을 조합해놓은 것들. combined, common, dev, short, tiny가 존재

7) cookie parser 미들웨어
--------------------

- 요청 쿠키를 추출하는 미들웨어
- request, response에 cookies 속성과 cookie() 메서드가 부여됨

```
$ npm install cookie-parser
```

- 기본적인 사용법 : cookie(이름,값,옵션)
- cookie() 메서드의 옵션
  - httpOnly : 클라이언트의 쿠키 접근 권한 지정
  - secure : https에서만 쿠키 사용
  - expries : Expires 속성 지정
  - maxAge : maxAge 속성 지정
  - path : 쿠키의 경로

```js
var express = require('express');

//서버 생성
var app = express();

//쿠키 미들웨어 설정
app.use(require("cookie-parser")());

app.get('/getCookie',function(request,response){
  response.send(request.cookies);
  });

app.get('/setCookie',function(request,response){
  response.cookie("string","cookie");
  response.cookie("json",{
    name:"cookie",
    id : "21lva"
    });
  response.cookie("json",{
    password:"1234"
    }, {
      //옵션 적용 가능
      maxAge : 3000,
      secure : true
      });
  reponse.redirect("/getCookie");
});

//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});
```

- 이렇게 설정된 쿠키는 클라이언트의 자바스크립트에서 사용가능하다

```js
//쿠키 가져오기
var getCookie = function(name) {
  var value = document.cookie.match('(^|;) ?' + name + '=([^;]*)(;|$)');
  return value? value[2] : null;
};

//쿠키 설정 : setCookie(변수이름, 변수값, 기간);
setCookie("expend", "true", 1);
```

8) body parser 미들웨어
-----------------

- POST 요청 데이터를 추출하는 미들웨어
- request 객체에 body 속성이 부여된다

```
$ npm install body-parser
```

- 다음은 POST 방식을 이용한 login이다

```js
var express = require('express');
var fs = require("fs");

//서버 생성
var app = express();

//쿠키 미들웨어 설정
app.use(require("cookie-parser")());

//body parser 미들웨어 설정
app.use(require("body-parser").urlencoded({extended:false}));

//라우터 설정
app.get('/',function(request,response){
  if(request.cookies.auth){
    response.send("<h1>Login Success</h1>");
  }
  else{
    response.redirect('/login');
  }
});
app.get('/login',function(request,response){
  //login page 출력
  fs.readFile("login.html",function(err,data){
    response.send(data.toString());
    });
});
app.post('/login',function(request,response){
  //쿠키 생성
  var loginId = request.body.loginId;
  var password = request.body.password;

  if(loginId==="abc"&& password==="1234"){
    response.cookie("auth",true);
    response.redirect("/");
  }
  else{
    response.redirect("/login");
  }

});
//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});
```

9) connect-multiparty 미들웨어
-----------------------------

- 파일은 다른 데이터보다 용량이 커서 multipart/form-data 인코딩 방식 사용
- connect-multiparty : 파일을 업로드할 때 사용. multipart/form-data 인코딩 방식을 사용할 수 있게 해준다.

```
$ npm install connect-mulitparty
```

- 다음은 upload.html 의 일부분이다

```html
<form method="post" enctype="multipart/form-data">
  <input type="text" name="comment"/>
  <input type='file' name='image'/.
  <input type="submit"/>
</form>
```

- 서버코드는 다음과 같다.

```js
var express = require('express');
var fs = require("fs");
var multiparty = require('connect-mulitparty');

//서버 생성
var app = express();

//multiparty 미들웨어 설정
app.use(multiparty({uploadDir:__dirname+"/multiparty"}));

//라우터 설정
app.get('/',function(request,response){
  //login page 출력
  fs.readFile("upload.html",function(err,data){
    response.send(data.toString());
    });
});

app.post('/',function(request,response){
  console.log(request.body);
  console.log(request.files);

  response.redirect("/");
});

//server 시작
app.listen(52273,function(){
  console.log('Server Running');
});
```

- 파일 속성은 files에 저장된다.
- 같은 사용자가 같은 방법으로 업로드하면 덮어씌우기가 된다.


- 특정 페이지에만 multiparty 미들웨어를 적용하고 싶으면 아래와 같이 한다

```js
app.post('/',multiparty,function(request,response){

});
```

- - -

ExpressJS 쉽게 사용하기
======================

- 사실, 웹서버의 모든 기능을 스스로 짜는 것은 힘들다
- express-generator를 이용하면 쉽게 웹페이지에 필요한 것들을 설치할 수 있다.

```
$ sudo npm install -g express-generator
```

- express 프레임워크를 쓸 폴더를 만들고, 폴더로 이동, 설치하자.

```
$ express --ejs exerciseExpress
$ cd exerciseExpress & npm install
```

- 시작하자. 기본 포트가 3000 이므로 127.0.0.1:3000 에 접속하면 볼 수 있다.

```
$ DEBUG=helloexpress:* npm start
```

1) 기본 프로젝트
--------------

- **express 프로젝트명** 으로 만든 기본 프로젝트에는 다음과 같은 폴더들이 존재한다
  - bin : 프로그램 실행과 관련된 파일이 들어있는 디렉토리. 이 디렉토리의 www 파일을 실행해서 프레임워크를 실행
  - node_modules :
  - public : express 모듈의 static 미들웨어를 사용해 웹 서버에 올라가는 폴더. js, css 등의 리소스 파일이 존재.
  - routes : page routing과 관련된 모듈. index.js, routes 파일이 존재
  - views : ejs(or jade) 파일과 같은 템플릿 파일 존재
  - app.js : 프로젝트의 내용과 관련된 것. 서버에 관련된 소스가 들어 있다.
  - package.json : 현재 프로젝트와 관련된 정보와 모듈을 설치하는 데 필요한 내용을 담고 있음.


- app.js 파일에는 서버 객체를 생성과 호출, 미들웨어, 라우팅을 설정하는 부분이 있다.
- 여기에 미들웨어를 추가할 수 있다.
- app.set() : express 프레임워크의 설정 옵션을 지정하는 메서드.

2) 페이지 렌더링
---------------

- express 프레임워크는 자체적으로 제공하는 render() 메서드를 사용하여 렌더링한다.
- views 디렉토리 안에 새로운 파일을 만들고(newP.ejs), app.js를 다음과 같이 고친다

```js
/*
코드
*/

var aRoute = require("./routes/newP");

/*
코드
*/

app.use("/newP",newPRoute);
```

- route폴더에 newP.js를 만들고 다음과 같이 작성한다.

```js
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('newP', { title: 'Express' });
});

module.exports = router;
```

- 127.0.0.1:3000/newP에 접속하면 페이지를 볼 수 있다.

3) 레이아웃 페이지
-----------------

- ejs로 만들 시 layout을 제공하지 않는다.
- ejs파일에 다음과 같이 layout을 적용시킬 수 있다.

```html
<% include headerLayout.ejs%>

/*
코드
*/

<% include footerLayout.ejs %>
```

- headerLayout.ejs 파일의 모습

```html
<DOCTYPE html>
<html>
<head>
  <title><%= title%></title>
</head>
<body>
  <header>
    <h1><%= title %></h1>
  </header>
```

- footerLayout.ejs도 비슷하다

```html
<footer></footer>
</body>
</html>
```

4) 실행 환경
------------

- development 환경과 production 환경이 존재.

||development|production|
|오류 출력| 출력| 출력X|
|view cache|사용X| 사용|
|디버그 모드|설정시 사용|사용X|

- view cache : 자주사용하는 ejs파일을 캐시에 저장하여 속도를 늘린다. 개발하는 동안에는 사용하지 말자(수정한 부분이 반영이 안될 수도 있음)

- 다음은 development환경과 production환경에서의 오류를 다루는 차이를 보여주는 app.js파일부분이다.

```js

/*
코드
*/

//404 error 부분

//development환경
///next가 없어서 아래의 production 환경에서의 오류 처리 미들웨어를 실행하지 않는다
if(app.get('env')==='development')
app.use(function(err,req,res,next){
  res.status(err.status||500);
  res.render('error',{
    messge:err.message,error:err
    });
  });

//production 환경
//error 내용을 출력하지 않는다
app.use(function(err,req,res,next){
  res.status(err.status||500);
  res.render('error',{
    messge:err.message,error:{}
    });
  });
```

- development는 기본이다. production 환경으로 실행하고 싶으면 다음과 같이 한다

```
//리눅스
$ export NODE_ENV=production & npm start

//윈도우
> SET NODE_ENV=production
> npm start
```

- 최종 배포를 할 경우에는 그냥 app.js에 다음과 같이 환경을 추가하는 것도 좋다

```js
var app = express();

app.setting.env= 'production';

/*
코드
*/
```
