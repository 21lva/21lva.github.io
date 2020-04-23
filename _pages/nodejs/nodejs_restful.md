---
layout: post
title:  "RESTful"
date:   2019-07-31 02:02:59
author: Inhyuk
categories: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_restful.md
---


RESTful
=======

- REST : REpresentational State Transfer의 약자
- 자원의 이름(표현)으로 구분하여 해당 자원의 정보를 주고 받는 것
- 웹의 기존 기술과 HTTP 프로토콜을 그대로 사용하기 때문에 웹의 장점을 최대화 할 수 있는 아키텍처 스타일
- HTTP URI(uniform resource identifier)을 통해 자원을 명시하고, http method(get,post,put,delete)를 통해 해당 자원에 대한 CRUD operation 적용
  - CRUD : create(생성,post), read(조회,get), update(수정,put), delete(삭제,delete)
  - URI : 웹서버가 리소스를 고유하게 식별할 수 있도록 하는 것. URL(uniform resource locator), URN 두 가지가 존재.

- REST 장점
  - HTTP 프로토콜을 사용 -> 별도의 구축이 필요 없음
  - 모든 플랫폼에서 사용 가능
  - 의도하는 바를 쉽게 파악
  - 서버와 클라이언틔 역할을 명확히 분리

- REST 단점
  - 표준이 없다
  - 사용할 수 있는 method의 종류가 제한적
  - 구형 브라우저에서는 지원하지 않는다

- 구성 요소
  - 자원(URI) : 모든 자원(서버에 존재)에는 ID가 존재. HTTP URI를 통해 구별(/groups/:group_id). 클라이언트가 서버에 해당 자원의 상태에 대한 조작을 요청
  - 행위(HTTP method)
  - 표현 : 클라이언트가 자원에 대한 조작을 요청하면 서버는 적절한 표현을 보낸다. JSON, XML, TEXT, RSS 등의 형태를 가진다.

- REST 특징
  - uniform : URI로 지정한 리소스에 대한 조작을 통일화
  - stateless : 무상태성(클라이언트의 context를 서버 쪽에서 유지하지 않는다). 세션과 쿠키를 저장하지 않는다.
  - Cacheable : HTTP가 가진 cache기능을 그대로 사용
  - self-descriptiveness : REST API 메세지만 보고도 쉽게 이해할 수 있는 구조
  - client-server architecture : 자원이 있는 쪽은 서버, 자원을 요청하는 쪽은 클라이언트. 클라이언트는 자신의 정보를 직접 관리. 의존성이 줄어든다
  - 계층형 구조 : 클라이언트는 REST API 서버만 호출. 서버는 다중 계층으로 구성될 수 있다.

|-|-|-|
|/collection|GET|컬랙션들의 목록 표시|
|/collection/:id|GET|컬랙션 중 하나를 표시|
|/collection|POST|컬랙션에 새로운 데이터 추가|
|/collection/:id|PUT|컬랙션의 데이터 수정|
|/collection/:id|DELETE|컬랙션의 데이터 삭제|

예제
----

```js
var fs = require('fs');
var express = require('express');
var bodyParser = require('body-parser');

//더미 DB 생성
var dummyDB = (function(){
  var storage =[];
  var count=1;
  var dDB={
    "get":function(id){
      if(id){
        id = typeof id=='string' ? Number(id):id;
        for(let i in storage)if(storage[i].id===id)return storage[i];
        return storage;
      }
    },
    "insert":function(data){
      data.id=count++;
      storage.push(data);
      return data;
    },
    "remove" : function(id){
      id = typeof id=='string' ? Number(id) : id;
      for(let i in storage)
        if(storage[i].id===id){
          storage.splice(i,1);
          return true;
        }
      return false;
    }
  };
  return dDB
})();

//서버 생성
var app = express();

//미들웨어 설정
app.use(bodyParser.urlencoded({extended:false}));

//라우터 설정
app.get('/user',function(request,response){
  //create
  response.send(DummyDB.get());
});
app.get('/user/:id',function(request,response){
  //read
  response.send(DummyDB.get(request.params.id));
});
app.post('/user',function(request,response){
  //insert
  var Name = request.body.name;
  var Region = request.body.region;

  if(name&&region){
    response.send(DummyDB.insert({name:Name,region:Region}));
  }
  else throw new Error('error');
});
app.put('/user/:id',function(request,response){
  //update
  var id = request.params.id;
  var Name = request.body.name;
  var Region = request.body.region;

  var item = DummyDB.get(id);
  item.name = Name|| item.name;
  item.region = Region|| item.region;

  response.send(item);
});
app.delete('/user/:id',function(request,response){
  //delete
  response.send(DummyDB.remove(request.params.id));
});

//실행
app.listen(52273,function(){
  console.log("Server Running at http://127.0.0.1:52273");
  });
```



- - -

참고자료
========

- [REST 자세히](https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html)
- [REST vs others](https://shlee0882.tistory.com/208)
- [URI](https://blog.lael.be/post/61)
