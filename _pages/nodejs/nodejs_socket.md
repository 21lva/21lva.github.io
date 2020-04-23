---
layout: post
title:  "Socket.io"
date:   2019-08-10 02:02:59
author: Inhyuk
category: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_socket.md
---

socket.io
==========

- socket.io : 웹소켓, 롱풀링 통신을 가능하게 해줌
- 웹소켓 : 서버와 클라이언틔의 양방향 연결 채널을 구성하는 HTML5 프로토콜. 특정 주기를 가지고 **polling** 하지 않아도 변경 사항을 전달.
- http는 양방향이 불가. 클라이언트가 **REQUEST** 를 보낼 때만 서버에서 **RESPONSE** 를 보낼 수 있음.
- Polling : 일정 시간 주기로 계속해서 물어(request)보는 것. 실시간이 불가능. 네트워크 부하가 올 수도 있음(setInterval 이용)
- Long Polling : 서버에서는 요청(request)를 받으면 일정 시간 동안 답변 보류 -> 업데이트 시에는 응답(response)를 보낸다 -> 대답을 받으면 다시 새로운 요청을 보낸다. 빈번한 데이터의 업데이트가 있으면 별 효과가 없다.


- 설치하자

```console
$ npm install socket.io@1
```

- 연습이기 때문에 간단하게 두 가지 파일(index.html, socket.io.server.js)을 준비하자


- socket.io.server.js

```js
var http = require("http");
var fs = require("fs");
var socketio = require("socket.io");

var server = http.createServer(function(req,res){
    fs.readFile('index.html',function(err,data){
        res.writeHead(200,{
            'Content-Type':'text/html'
        });
        res.end(data);
    });
}).listen(52273,function(){
    console.log("Sever running at http://127.0.0.1:52273");
});

//socket 서버 생성, 실행
var io = socketio.listen(server);
io.sockets.on('connection',function(socket){

});
```

- index.html의 모습.
- 웹서버에 socket.io.js 파일이 없는 데도 사용가능하다(socket.io 모듈을 사용하면 자동으로 socket.io.js파일이 등록됨)

```js
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Socket.io</title>
    <script src='/socket.io/socket.io.js'></script>
    <script>
    window.onload = function(){
        var socket=io.connect();
    };
    </script>
</head>
<body>

</body>
</html>
```

- 디버그 모드로 실행하고 브라우저로 접속하자

```console
$ export DEBUT=socket.io*
$ node socket.io.server.js
```

- 다음과 같이 접속되었음을 알 수 있다

```console
socket.io:server initializing namespace / +0ms
socket.io-parser encoding packet {"type":0,"nsp":"/"} +0ms
socket.io-parser encoded {"type":0,"nsp":"/"} as 0 +1ms
socket.io:server creating engine.io instance with opts {"path":"/socket.io","initialPacket":["0"]} +6ms
socket.io:server attaching client serving req handler +19ms
Sever running at http://127.0.0.1:52273
socket.io:server serve client source +6s
socket.io:server incoming connection with id GFK0N5ZqfrI84Y9VAAAA +40ms
socket.io:client connecting to namespace / +0ms
socket.io:namespace adding socket to nsp / +0ms
socket.io:socket socket connected - writing packet +0ms
socket.io:socket joining room GFK0N5ZqfrI84Y9VAAAA +0ms
socket.io:socket packet already sent in initial handshake +1ms
socket.io:socket joined room GFK0N5ZqfrI84Y9VAAAA +2ms
```

1) 이벤트
---------

- socket.io도 이벤트 리스너 등록, 실행을 **on(), emit()** 메서드로 진행한다.
- 연결 이벤트
1. connection : 클라이언트가 연결할 때 발생
1. disconnect : 클라이언트가 연결을 끊을 때 발생


- 이벤트를 다음과 같이 만들어보자.
1. 클라이언트에서 rint이벤트가 발생하면 캐치해서 콘솔에 정보를 띄운다
2. smart이벤트를 발생시키고 클라이언트에 전달한다.

```js
var io = socketio.listen(server);
io.sockets.on('connection',function(socket){
  socket.on("rint",function(data){
    console.log("Client Send : ",data);
    socket.emit("smart",data);
    });  
});
```

- html 파일도 수정하자
1. 서버에서 보내는 smart 이벤트를 받을 이벤트 리스너를 등록한다
2. 버튼을 눌렀을 때, 값을 입력 받아서 서버에 전달(rint이벤트)할 리스너를 등록한다.

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Socket.io</title>
    <script src='/socket.io/socket.io.js'></script>
    <script>
    window.onload = function(){
        var socket=io.connect();
        socket.on('smart',function(data){
          alert(data);
          });
        document.getElementById("button").onclick = function(){
          var text = document.getElementById("text").value;
          socket.emit("rint",text);
        };
    };
    </script>
</head>
<body>
  <input type="text" id="text"/>
  <input type="button" id="button"/>
</body>
</html>
```

2) 소켓 통신 종류
----------------

- public : 자신(클라이언트)을 포함한 모든 클라이언트에 데이터 전달
- broadcast : 자신(클라이언트)을 제외한 모든 클라이언트에 데이터 전달
- private : 특정 클라이언트에 데이터를 전달


```js
//public
io.sockets.on('connection',function(socket){
  socket.on("rint",function(data){
    io.socket.emit("smart",data);
    });  
});

//broadcast
io.sockets.on('connection',function(socket){
  socket.on("rint",function(data){
    socket.broadcase.emit("smart",data);
    });  
});

//private
var id =0;
io.sockets.on('connection',function(socket){
  //특정 id설정. 가장 최근에 접속한 사람에게 보낸다
  id = socket.id;

  socket.on("rint",function(data){
    io.socket.to(id).emit("smart",data);
    //to() 대신 in() 사용 가능
    });  
});
```

3) 방 생성
-----------

- socket.io는 방(room,group)을 생성하는 기능이 존재
1. socket.join() : 클라이언트를 방에 추가
2. io.sockets.in() /io.sockets.to() : 특정 방에 있는 클라이언트 추출

- index.html과 socket.room.js 파일을 준비하자
- js파일의 모습

```js
var http = require("http");
var fs = require("fs");
var socketio = require("socket.io");

var server = http.createServer(function(req,res){
    fs.readFile('index.html',function(err,data){
        res.writeHead(200,{
            'Content-Type':'text/html'
        });
        res.end(data);
    });
}).listen(52273,function(){
    console.log("Sever running at http://127.0.0.1:52273");
});

//socket 서버 생성, 실행
var io = socketio.listen(server);
io.sockets.on('connection',function(socket){
  var roomName = null;

  //join event
  socket.on('join',function(data){
    roomName = data;
    socket.join(data);
    });

  //message event
  socket.on('message',function(data){
    //public
    io.sockets.in(roomName).emit('message','test');
    });
});
```

- index.html 의 모습

```js
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Socket.io</title>
    <script
			  src="https://code.jquery.com/jquery-3.4.1.js"
			  integrity="sha256-WpOohJOqMqqyKL9FccASB9O0KwACQJpFTUBLTYOVvVU="
			  crossorigin="anonymous"></script>
    <script src='/socket.io/socket.io.js'></script>
    <script>
    window.onload = function(){
        var room = prompt('Please type Room name'.'');
        var socket=io.connect();

        socket.emit('join',room);
        socket.on('message',function(data){
          $('<p>'+data+'</p>').appendTo('body');
          });
        $('#button').onclick =function(){
          socket.emit('message','socket.io room message');
        };
    };
    </script>
</head>
<body>
  <input type="text" id="text"/>
  <input type="button" id="button"/>
</body>
</html>
```

4) 웹 채팅 프로그램
--------------

- 사용하는 모듈 : socket.io, http, fs
- 구성 : app.js, index,html

- 웹 채팅 서버 : 2가지 이벤트
1. 클라이언트에서 메세지를 입력하면 서버로 데이터(이름, 메세지, 시간) 전달.
2. 서버에서 받은 정보를 가지고 다른 클라이언트(public 방식)에 데이터 전달

```js
var http = require("http");
var fs = require("fs");
var socketio = require("socket.io");

var server = http.createServer(function(req,res){
    fs.readFile('index.html',function(err,data){
        res.writeHead(200,{
            'Content-Type':'text/html'
        });
        res.end(data);
    });
}).listen(52273,function(){
    console.log("Sever running at http://127.0.0.1:52273");
});

//socket 서버 생성, 실행
var io = socketio.listen(server);
io.sockets.on('connection',function(socket){
  //message event
  socket.on('message',function(data){
    //public
    io.sockets.in(roomName).emit('message','test');
    });
});
```

- 웹 채팅 클라이언트

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Socket.io</title>
    <script
			  src="https://code.jquery.com/jquery-3.4.1.js"
			  integrity="sha256-WpOohJOqMqqyKL9FccASB9O0KwACQJpFTUBLTYOVvVU="
			  crossorigin="anonymous"></script>
    <script src='/socket.io/socket.io.js'></script>
    <script>
    window.onload = function(){
        var room = prompt('Please type Room name'.'');
        var socket=io.connect();
        socket.on('message',function(data){
          var output ='';
          output+='<li>';
          output+='<h3>'+data.name+</h3>;
          output+='<p>'+data.message+</p>;
          output+='<p>'+data.date+</p>;
          output+='</ li>';
          $(output).appendTo('#content');
          });
        $('button').onclick =function(){
          socket.emit('message',{
            name:$('#name').val(),
            message:$('#message').val(),
            date:new Date().toUTCString()
            });
        };
    };
    </script>
</head>
<body>
  <h1>Socket.io Chatting</h1>
  <p>Chat With Node.js</p>
  <hr/>
  <input type="text" id="name"/>
  <input type = "text" id ="message"/>
  <button>Send!</button>
  <ul id= "content">
  </ul>
</body>
</html>
```
- - -

ExpressJS와 함께 사용하기
========================

- express-generator를 이용하여 프레임을 만든 서버에서는 **www** 와 **app.js** 를 수정해줘야 한다
- 여기서는 socket.io관련 js파일은 **routes/chat/chat.js** 에 설정

#### routes/chat/chat.js

```js
var express = require('express');
var router = express.Router();

//socket io  setup
var io=require("socket.io")();

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

    // 메시지를 전송한 클라이언트를 제외한 모든 클라이언트에게 메시지를 전송한다
    //socket.broadcast.emit('chat', msg);

    // 메시지를 전송한 클라이언트에게만 메시지를 전송한다
    // socket.emit('chat', msg);

    // 접속된 모든 클라이언트에게 메시지를 전송한다
    io.emit('chat', msg);

    // 특정 클라이언트에게만 메시지를 전송한다
    // io.to(id).emit('chat', data);
  });
});

module.exports = io;
```

#### app.js

```js
//생략

var app = express();

//socket.io router
app.io = require("./routes/chat/chat");

//생략
```


#### bin/www

```js
//생략
var server = http.createServer(app);

// app.io에 server를 추가한다
app.io.attach(server);

//생략
```
- - -

참고자료
=======

- <https://poiemaweb.com/nodejs-socketio>
