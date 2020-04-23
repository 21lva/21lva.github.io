---
layout: post
title:  "기본 기능"
date:   2019-07-28 02:02:59
author: Inhyuk
category: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_basic.md
---

[Do it! Node.js 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=11738465)를 공부한 것을 정리하였다.

- **Node.js** : 자바스크립트를 기반으로하여 서버를 동작하게 해주는 엔진. 비동기 방식 + 이벤트 기반의 입출력 모델의 방식 사용. 크롬 V8엔진 위에서 실행된다.
- 이벤트 기반 : 특정 이벤트가 발생할 때 무엇을 할지(callback)를 등록.

- - -

전역객체와 모듈
========

- console : 콘솔 창에 결과를 보여주는 객체
- process : 프로세스의 실행에 대한 정보를 다루는 객체
- exports : 모듈을 다루는 객체

1) console
-----------

```js
console.log("숫자 : %d / 문자열 : %s / 객체 : %j",10,"hi",{name:"hello"});
console.dir(object) //객체의 속성 출력
console.time(id) // 실행시간 측정을 위한 시작 시간 출력
console.timeEnd(id) // 실행시간 측정을 위한 끝나는 시간 출력
```

2) process
-----------

- 프로그램을 실행했을 때의 프로세스 정보를 다룬다
- argv : 프로세스에 전달되는 매개변수 정보
- env : 환경 변수
- exit() : 프로세스 종료

```js
console.log(process.env);
console.dir(process.argv);
```

3) exports
----------

- 모듈을 사용할 수 있게 해주는 전역객체

```js
//module.js
exports.mul = function(a,b){
  return a*b;
}
```

```js
// main.js
var mod1 = require("module");
mod1.mul(1,2);
```

4) 내장 모듈
------------

- os : 시스템 정보를 알려주는 모듈.
  - hostname() : 운영체제의 호스트 이름
  - totalmem() : 시스템 전체의 메모리 용량
  - freemem() : 사용 가능한 메모리 용량
  - cpus() : cpu 정보
  - networkInterfaces() : 네트워크 인터페이스를 담은 배열

- path : 파일 경로를 다루는 모듈
  - join() : 여러개의 이름을 합쳐서 하나의 파일 경로를 만든다
  - dirname() :디렉토리 이름 반환
  - basename() : 확장자 이름을 제외한 파일 경로 반환
  - extname() : 파일 확장자를 반환

```js
var path = require("path");
var docsDir = path.join("/Users","a.exe");
var filename = "C:\\Users\\fo\\a.exe";
var basename = path.basename(filename);
var extname = path.extname(filename);
```

- - -

기본 기능
=========

1) 주소 문자열
------------

- url 모듈 : 웹페이지의 url을 다루는 모듈
  - parse() : 주소문자열을 파싱하여 url 객체 생성
  - format() : URL 객체를 주소 문자열로 파싱

- querystring 모듈 : 요청 parameter를 분류하는 데에 사용
  - parse() : 요청 parameter를 파싱하여 객체로 생성
  - stringify() : 객체를 문자열로 변환

```js
var url = require("url");
var qS = require("querystring");

//url 객체 만들기
var curURL = url.parse("https://www.google.com/search?q=%EB%82%98%EB%AC%B4%EC%9C%84%ED%82%A4");

// url 객체를 주소 문자열로 파싱
var curStr = url.format(curURL);

//요청 parameter 구분
var parameters = qS.parse(curURL.query);

console.dir(curURL);
console.log(parameters.query);
console.log(qS.stringify(parameters));
```

2) 이벤트
---------

- 노드는 이벤트 + 비동기 기반.
- 한쪽에서 이벤트를 발생시키면 이벤트 리스너가 이를 확인하여 절차를 수행.
- EventEmitter : 이벤트를 보내고 받도록 사용하는 것.
1. on(event,listener) : 이벤트 리스너를 추가
2. once(event,lisnter) : 이벤트 리스너를 추가. But 한번 이벤트 발생시 이벤트 리스너 제거.
3. removeListener(event,listener) : 이벤트 리스너 제거.
4. emit(event,parameters) : 이벤트를 다른 쪽으로 전달

- process객체는 EventEmitter를 상속 받고 있다.

```js
//기존 이벤트에 리스너 등록
process.on("exit",function(){
  console.log("EXIT");
  });

//emit을 이용한 다른 이벤트 리스너 등록
process.on("myEvent",function(a,b){
  console.log("This is my custom event. Parameters : %d %d",a,b);
  });

//이벤트 발생
process.emit("myEvent",1,2);

setTimeout(function(){
  console.log("2초후 종료");
  },2000);
process.exit();
```

- EventEmitter를 직접 상속 받아 이벤트 등록을 하기를 원한다면 다음과 같이 한다.

```js
var util = require("util");
var EE = require("events").EventEmitter;

var Obj = function(){
  var a =1;
  this.on("eventOccur",function(){
    console.log("A : %d",a);
    });
}

util.inherits(Obj,EE);
var OB = new Obj;

OB.emit("eventOccur");
```

3) 파일 다루기
----------

- 동기식 IO : 파일 작업이 끝날 때 까지 기다린다. Sync라는 단어가 붙는다.
- 비동기식 IO : 파일 작업을 요청하고 다음 작업을 수행할 수 있다.
- fs 모듈 : 파일 IO에 사용되는 모듈
1. readFile(filename,[encoding],[callback]) : 비동기식으로 파일을 읽는다.
2. readFileSync(filename,[encoding]) : 동기식으로 파일을 읽는다.
3. writeFile(filename,data,encoding="utf8",[callback]) : 비동기식으로 파일을 쓴다. 에러 발생시 callback으로 에러(null이 아닌 값)을 전달한다.
3. writeFileSync(filename,data,encoding="utf8") : 비동기식으로 파일을 쓴다.

- 파일을 전체를 열고 읽거나 쓰는것이 아니라 다양한 방법이 필요할 때도 있다.
1. open(path,flags,[mode],[callback]) : 파일을 연다.
1. read(fd,buffer,offset,length,position,[callback]) : 지정된 부분의 파일을 읽는다. buffer에 저장한다.
1. write(fd,buffer,offset,length,position,[callback]) : 지정된 부분의 파일에 buffer를 쓴다.
1. close(fd,[callback]) : 파일을 닫는다.


```js
var fs=require("fs");

//Read
fs.open("./output.txt","w",function(err,fd){
  if(err){
    console.log("Error Occured");
    return;
  }
  var buf = new Buffer("hi");
  fs.write(fd,buf,0,buf.length,null,function(err,written,buffer){
    if(err)throw err;
    console.log(err,written,buffer);  

    fs.close(fd,function(){
      console.log("File closed");
      });
    });
  });

//Write
fs.open("./output.txt","r",function(err,fd){
  if(err){
    console.log("Error Occured");
    return;
  }
  var buf = new Buffer(15);
  fs.read(fd,buf,0,buf.length,null,function(err,byteRead,buffer){
    if(err)throw err;
    var bufStr = buffer.toString("utf8",0,byteRead);
    console.log("Read : %s",bufStr);  
    console.log(err,byteRead);  

    fs.close(fd,function(){
      console.log("File closed");
    });
  });
});
```

- 파일의 크기가 크면 스트림을 이용하여 데이터를 처리할 수 있다.
1. createReadStream(path,[options]) : 파일을 읽기 위한 stream 객체 생성
1. createWriteStream(path,[options]) : 파일을 쓰기 위한 stream 객체 생성

```js
var fs = require("fs");

var rFile = fs.createReadStream("output.txt",{flgas:"r"});
var oFile = fs.createWriteStream("output2.txt",{flgas:"a"});

rFile.on("data",function(data){
  console.log("Read : ",data);
  ofile.write(data);
  });

rFile.on("end",function(){
  console.log("Read completed");
  oFile.end(function(){
    console.log("Write completed");
    });
  });
```

- 또는 간단하게 pipe()를 이용하여 둘을 연결할 수도 있다.

```js
var fs = require("fs");

var rFile = fs.createReadStream("output.txt",{flgas:"r"});
var oFile = fs.createWriteStream("output2.txt",{flgas:"a"});

rFile.pipe(oFile);
```

- 스트림 방법은 웹서버에서 사용자의 요청을 처리할 때에 사용될 수도 있다

```js
var fs = require("fs");
var http = require("http");

var server = http.createServer(function(req,res){
  var inStream = fs.createReadStream("output.txt");  
  inStream.pipe(res);
  });
server.listen(7001,"127.0.0.1");
```

- fs모듈로 디렉토리를 생성하고 삭제할 수도 있다.

```js
fs.mkdir(디렉토리 이름, 권한,[callback]);
fs.rmdir(디렉토리 이름,[callback]);
```

4) 로그
--------

- winston 모듈 : 로그를 남기는 모듈.

```js
var winston = require('winston');
var winstonDaily = require('winston-daily-rotate-file');
var moment = require('moment');

function timeStampFormat() {
    return moment().format('YYYY-MM-DD HH:mm:ss.SSS ZZ');                            
};
//logger 설정
var logger = new (winston.Logger)({
    transports: [
        new (winstonDaily)({
            name: 'info-file',
            filename: './log/server',
            datePattern: '_yyyy-MM-dd.log',
            colorize: false,
            maxsize: 50000000,
            maxFiles: 1000,
            level: 'info',
            showLevel: true,
            json: false,
            timestamp: timeStampFormat
        }),
        new (winston.transports.Console)({
            name: 'debug-console',
            colorize: true,
            level: 'debug',
            showLevel: true,
            json: false,
            timestamp: timeStampFormat
        })
    ],
    exceptionHandlers: [
        new (winstonDaily)({
            name: 'exception-file',
            filename: './log/exception',
            datePattern: '_yyyy-MM-dd.log',
            colorize: false,
            maxsize: 50000000,
            maxFiles: 1000,
            level: 'error',
            showLevel: true,
            json: false,
            timestamp: timeStampFormat
        }),
        new (winston.transports.Console)({
            name: 'exception-console',
            colorize: true,
            level: 'debug',
            showLevel: true,
            json: false,
            timestamp: timeStampFormat
        })
    ]
});

//logger를 통한 로그 출력
logger.info("infolevel 로깅");
```

- winston-daily-rotate-file : 매일매일 날짜별로 파일을 만들어 저장을 하게 해주는 모듈
- moment :
- 로그의 파일은 filename+dataPattern으로 만들어진다. 여기서는 실행한 파일의 경로에 log 디렉토리를 만들고 "server_2019_09_29.log" 와 같이 로그를 작성한다.
- 한 파일의 최대 크기는 maxsize, 파일의 최대 갯수는 maxFiles이다.
- level : 로그를 작성하는 경우이다. debug(0) > info(1) > notice(2) > warning(3) > error(4) > crit(5) > alert(6) > emerg(6) 순으로 많은 내용을 포함한다.
- showLevel : 로그를 적은 level을 보여준다.
- timestamp : 콜백함수를 적는다. 로그를 출력하는 데에 필요한 시간 포맷을 전달하는 함수이다.

- - -

참고자료
=======

- [EventEmitter](https://dololak.tistory.com/89)
- [Logging](https://dololak.tistory.com/126?category=642882)
