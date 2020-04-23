---
layout: post
title:  "Redis"
date:   2019-08-10 02:02:59
author: Inhyuk
categories: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_redis.md
---


redis
=====

- 메모리에 저장하는 key-value 형태의 NoSQL
- nodejs에서 사용하는 redis 모듈 [관련문서](https://www.npmjs.com/package/redis)

- 설치
```
sudo npm install JSON redis -g
```

- 기능
  - set key value: key-value 저장
  - get key : key-value의 value 읽기
  - publish : 특정 사안을 subscribe하는 client에게 메시지 전달
  - subscribe : 특정 사안을 구독

- 자주 사용되는 구조
  - String : key에 대해서 문자열을 저장. 최대 512MB의 바이너리도 저장가능
  - List : key에대해서 linked list 자료 구조를 매칭. lpush(처음에 저장), rpush(마지막에 저장), lpop(처음 뽑기), rpop(마지막 뽑기) 가능
  - Set : List와 달리 value의 중복 불가능
  - Sorted Set : set과 비슷. value와 함께 score 저장. score를 기준으로 정렬
  - Hash : value에 객체 형식을 저장(field-value의 여러쌍이 value에 들어간다)

```js
//client 생성
var client = require('redis').createClient();

//String
client.set('key1','value1');
client.get('key1',(err,value)=>{
  console.log(value);
});

//List
client.rpush('key','value1','value2','value3');
client.lpush('key','value1','value2','value3');
client.lrange('key',0,-1,(err,arr)=>{
  console.log(arr)  
});

//Set
client.sadd('key','value1','value2','value3');
client.srange('key',(err,s)=>{
  console.log(s);
});

//Sorted Set
client.zadd('key',score1,'value1',score2,'value2',score3,'value3');
client.zrange('key',0,-1,(err,s)=>{
  console.log(s);
});

//Hash
client.hmset('key','field1','value1','field2','value2');
client.hgetall('key',(err,obj)=>{
  console.log(obj);
});

//Delete
client.del('key');

//exist
client.exists('key');

//rename
client.rename('old_key','new_key');
```
- - -

Socket.io + Redis 로 채팅서버 구현
================================

- 이전 장인 Socket.io를 바탕으로 진행

#### chat.js

```js
var express = require('express');
var redis = require('redis');
var JSON = require('JSON');
var router = express.Router();

//create socket.io
var io=require("socket.io")();

//create redis client
var rclient = redis.createClient(6379,'127.0.0.1');


io.on("connection",(socket)=>{
  //login handler
  socket.on('login',(data)=>{
    console.log('Client logged in. name:'+data.name);
    socket.name=data.name;
    io.emit('login',data.name);
  });

  //chat handler
  socket.on('chat', function(data) {
    console.log('Message from %s: %s', socket.name, data.msg);

    var msg = {
      from: {
        name: socket.name,
      },
      msg: data.msg
    };

    //push msg to list
    rclient.rpush('chatLogs',JSON.stringify(msg));

    io.emit('chat', msg);

  });
});

module.exports = io;
```

#### index.js

```js
var express = require('express');
var router = express.Router();
var redis = require('redis');


var rclient = redis.createClient(6379,'127.0.0.1');

/* GET home page. */
router.get('/', function(req, res, next) {
  //load chat logs
  rclient.lrange('chatLogs',0,-1,(err,charArr)=>{//0 ~ -1(last element)
    let chatLogs=charArr.map(char=>JSON.parse(char));
    res.render('index', {
     title: 'Express',
     chatLogs:JSON.stringify(chatLogs)
    });
  });
});

module.exports = router;
```

#### index.ejs

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Socket.io Chat Example</title>
</head>
<body>
  <div class="container">
    <h3>Socket.io Chat Example</h3>
    <form class="form-inline">
      <div class="form-group">
        <label for="msgForm">Message: </label>
        <input type="text" class="form-control" id="msgForm">
      </div>
      <button type="submit" class="btn btn-primary">Send</button>
    </form>
    <div id="chatLogs">
    </div>
  </div>
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.2.4/jquery.min.js"></script>
  <script src="/socket.io/socket.io.js"></script>
  <script>
  $(function(){
    // socket.io 서버에 접속한다
    var socket = io();

    // 서버로 자신의 정보를 전송한다.
    socket.emit("login", {
      // name: "ungmo2",
      name: 'abc',
    });

    // 서버로부터의 메시지가 수신되면
    socket.on("login", function(data) {
      $("#chatLogs").append("<div><strong>" + data + "</strong> has joined</div>");
    });

    // 서버로부터의 메시지가 수신되면
    socket.on("chat", function(data) {
      $("#chatLogs").append("<div>" + data.msg + " : from <strong>" + data.from.name + "</strong></div>");
    });

    // Send 버튼이 클릭되면
    $("form").submit(function(e) {
      e.preventDefault();
      var $msgForm = $("#msgForm");

      // 서버로 메시지를 전송한다.
      socket.emit("chat", { msg: $msgForm.val() });
      $msgForm.val("");
    });
    var chatLogs = <%- chatLogs %>;
    chatLogs.forEach((chatLog)=>{
      $("#chatLogs").append("<div>" + chatLog.msg + " : from <strong>" + chatLog.from.name + "</strong></div>");
    });
  });
  </script>
</body>
```

- view 페이지에 JSON을 넘길 때 조심할 점
  - 서버에서 JSON을 넘길때는 stringify로 문자열화 하자
  - ejs의 경우 view 페이지에서 **<%=** 대신 **<%-** 를 사용하자


참고자료
=======

- <https://bcho.tistory.com/1098>
- <http://library.gabia.com/contents/infrahosting/8018>
- <https://wi-fi.tistory.com/entry/%EC%8B%A4%EC%8B%9C%EA%B0%84-%EC%B1%84%ED%8C%85%EB%B0%A9-%EB%A7%8C%EB%93%A4%EA%B8%B0-NodeJS-Express-Redis-Socketio-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0>
- <https://www.zerocho.com/category/NodeJS/post/5a3238b714c5f9001b16c430>
