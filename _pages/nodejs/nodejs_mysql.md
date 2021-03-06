---
layout: post
title:  "mysql"
date:   2019-07-31 02:02:59
author: Inhyuk
category: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_mysql.md
---

mysql 모듈
==========

- 설치하자

```console
$ npm install mysql
```

1) connection 객체
-------------------

- createConnection(options) : 데이터베이스에 접속. connection 객체 생성
1. host : 연결 호스트
2. port : 연결할 포트
3. user : 사용자 이름(필수)
1. password : 사용자 비밀번호(필수)
1. database : 연결할 데이터베이스
1. debug : 디버그 모드를 할지 말지

- query(sql[,callback]) : connection 객체의 메서드. 쿼리 문장을 실행. 결과는 콜백 함수의 두 번째 매개변수로 들어간다.

```js
var mysql = require('mysql');

var client = mysql.createConnection({
  user:"root",
  password:"1234",
  port :"3310"
  });

client.query("USE company");
client.query("SELECT * FROM products",function(error,result,fields){
  if(error)console.log("Query error");
  else console.log(result);
});
```

2) CRUD 구현
-------------

```js
var fs = require('fs');
var ejs = require('ejs');
var mysql = require('mysql');
var express = require('express');
var bodyParser = require('body-parser');

//DB와 연결
var client = mysql.createConnection({
  user:"root",
  password:"1234",
  port :"3310",
  database:"company"
});

//서버 생성
var app = express();
app.use(bodyParser.urlencoded({extended:false}));

//서버 실행
app.listen(52273,function(){
  console.log('Server Running at http://127.0.0.1:52273');
});


//조회
app.get("/",function(req,res){
  fs.readFile("list.ejs","utf8",function(err,data){
    //select를 시행하고 이 데이터를 ejs 파일에 넣어 출력한다.
    client.query("select * from products",function(err,result){
      res.send(ejs.render(data,{
        data:results
        }));
      });
    });
});

//삭제
app.get("/delete/:id",function(req,res){

});

//삽입
app.get("/insert",function(req,res){
  fs.readFile('insert.html',"utf8",function(err,data){
    res.send(data);
    });
});
app.post("/insert",function(req,res){
  var body = req.body;

  client.query("insert into products (name,modelnumber,series) values (?,?,?)",[body.name,body.modelnumber,body.series],function(){
    res.redirect("/");
    });
});

//수정
app.get("/edit/:id",function(req,res){
  //숫자를 입력하면(or 누르면) 미리 입력된 정보로 페이지 작성
  fs.readFile("edit.ejs","utf8",function(err,data){
    client.query('select * from products where id = ?',[req.params.id],
    function(err,result){
      res.send(ejs.render(data,{data:result[0]}));
      });
    });
});
app.post("/edit/:id",function(req,res){
  //보여준 값을 수정하고 제출 하면 DB를 수정한다
  var body = req.body;
  client.query("update products set name=?, modelnumber=?, series=? where id=?",[body.name,body.modelnumber,body.series,req.params.id],function(){
    res.redirect('/');
    });
});
```

(왜 put, delete를 안쓰는 것일까..)
