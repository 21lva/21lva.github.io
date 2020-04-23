---
layout: post
title:  "인증"
date:   2019-08-10 02:02:59
author: Inhyuk
category: NodeJS
tags: NodeJS
cover:  "/assets/instacode.png"
name: nodejs_auth.md
---

인증
====

- 모든 API 요청에 대한 사용자를 확인하는 작업
- HTTP요청에 대하여 사용자에 맞게 응답을 해야 함
- 각각의 HTTP의 헤더에 인증수단을 보내서 요청을 진행

- - -

Session, Cookie 방식
===================

- **Session** 은 서버에 저장하는 데이터이다
- **Cookie** 는 클라이언트에 저장하는 데이터이다
- 두 방식을 이용하는 인증은 다음의 순서로 진행된다
  1. 사용자가 인증요청을 서버에 보냄
  2. DB에서 사용자를 확인
  3. 사용자의 고유한 ID값을 이용하여 세션저장소(redis)에 저장
  4. 세션ID를 서버에 부여
  5. 사용자는 세션 ID를 쿠키에 저장. 필요할 때마다 사용
  6. 서버에서 사용자가 보낸 쿠키와 세션을 비교하여 인증 진행

- 쿠키에는 세션ID가 저장되지만, 사용자의 중요 정보를 가지고 있지는 않음. 즉, 이 쿠키가 탈취되어도 시간이 흐르면 다시 이 사용자의 정보를 이용할 수 없다
- 사용자로부터 쿠키를 확인하므로 사용자를 인증하기 위하여 DB에 접근할 필요 없음

<!--
예제
-->

- - -

JWT(JSON Web Token)
==================

- [JWT에 관한 글](https://velopert.com/2350)
- 인증에 필요한 정보를 암호화 시킨 토큰을 의미
- 사용자는 Access Token을 HTTP header에 넣어서 전송
- Token 구조 : **Encoded_header.Encoded_payload.Verify_signature**
  1. Header : 암호화 방식, 타입 등
  2. Payload : 서버에서 보낼 데이터. 사용자 id, 유효기간 등이 들어감
  3. Verify_signature :  base64 방식으로 인코딩한 Header, Payload, Secret key를 더한 후 서명
- Verify_signature는 Secret key를 알지 못하면 복호화 불가능
- x가 y의 데이터를 보기 위해 자기 token의 payload 부분을 y의 것으로 바꾼다고 하더라도, 서버에서 Verify_signature를 복호화하면 x에 대한 정보가 token에 나오므로 조작된 token이라고 인식 가능

```js
const crypto = require('crypto');
const verifySignature = crypto.createHmac('sha256','secret')
                          .update(encodedHeader+"."+encodedPayload)
                          .diges('base64')
                          .replace('=','');
```

- 인증 절차
  1. 사용자 로그인
  2. 서버에서 DB확인 후 사용자id + 만료기간을 이용하여 payload 생성
  3. secret key를 이용하여 access token 생성
  4. 사용자는 access token을 받아 저장한 후 인증이 필요할 때마다 전송
  5. 서버에서는 사용자가 보낸 access token을 verify signature를 이용하여 복호화 한 후에 사용자가 맞는지 or 만료기간이 지났는지 확인
  6. 인증 후에 payload를 디코딩하여 사용자의 id에 맞는 절차 진행

- 장점
  - 별도의 저장소 필요 없음(stateless)
  - Token 기반의 다른 인증 시스템과 쉽게 연동 가능
- 단점
  - 이미 발급된 JWT를 취소 시키는 방법이 없음(Refresh token으로 어느 정도 해결)
  - payload에 넣을 정보가 제한적
  - JWT의 길이가 길어지면 자원낭비가 발생함

예제
-----

- 간단한 jwt를 사용하는 것을 해보도록 하자

#### (1) 필요한 모듈 설치

```
npm install --save jsonwebtoken
npm install --save mysql
```

#### (2) index.js

```js
var express = require('express');
var router = express.Router();

var crypto = require('crypto');
var jwt = require('jsonwebtoken');

var client = require('mysql').createConnection({
  host:'localhost',
  user:'root',
  password:'1234',
  database:'toyDB'
});

const cryptoConfig ={
  "iteration":100000,
  "keylen":8,
  "digest":'sha512'
}

const secret = '!#@$@F@#!@';

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index');
});

router.post('/signup',(req,res,next)=>{
  const uid = req.body.id;
  const password = req.body.password;
  client.query('select * from Usr where uid=?',[uid],(err,rows)=>{
    if(err)throw err;
    if(rows[0])return res.send("NEW ID PLEASE");

    //make encoded password
    const salt= crypto.randomBytes(8).toString('hex');
    const key = crypto.pbkdf2Sync(password,salt,cryptoConfig.iteration,cryptoConfig.keylen,cryptoConfig.digest).toString('hex');

    //save id, encoded password with salt
    client.query('insert into Usr(uid,password,salt) values(?,?,?)',[uid,key,salt],(err,result)=>{
      if(err)throw err;
      return res.redirect('/');
    });
  });
});

router.post('/login',(req,res,next)=>{
  const uid = req.body.id;
  const password = req.body.password;

  //jwt is saved as a cookie
  const userToken = req.cookies.userToken;
  try{
    //if verify is failed because of some reasons like expiration, then it returns an error
    let dcd = jwt.verify(userToken,secret);
    return res.send("login success with a token");
  }catch(err){
    //make new token.
    const newToken = jwt.sign({"id":uid},secret,{expiresIn:30});
    client.query("select * from Usr where uid=?",[uid],(err,rows)=>{
      if(err)console.log(err);
      if(!rows[0])return res.send("NO MATCHED USER");//no matched user
      const derivedKey= crypto.pbkdf2Sync(password,rows[0].salt,cryptoConfig.iteration,cryptoConfig.keylen,cryptoConfig.digest).toString('hex');
      if(derivedKey===rows[0].password){
        //save new token as a cookie
        res.cookie('userToken',newToken);
        return res.send("login success with new token");
      }
      res.send("Wrong Password");
    });
  }
});
module.exports = router;
```

Refresh Token
--------------

- Access token만 이용하는 방식의 문제점은 유효기간이 길면 이 token이 탈취 되었을 때 보안이 취약해지고, 유효기간이 짧으면 빈번한 로그인을 해야한다는 것
- 이를 **유효기간이 짧은 Access Token + 유효기간이 긴 Refresh Token** 으로 어느 정도 해소
- access token이 만료되면 refresh token을 확인하고, 인증이 되면 access token을 재발급(로그인 불필요)
- refresh token이 만료되면 재 로그인 해야함(refresh token이 탈취되었을 때의 문제점 보완)

- 인증과정
  1. 사용자 첫 로그인
  2. 서버에서 사용자 인증 후 Access token, Refresh token 발급
  3. 사용자는 Access token의 만료전까지 Access token만 요청에 사용
  4. Access token이 만료되면, 서버에서 사용자에게 만료 신호 전송
  5. 사용자는 Refresh token 전송.
  6. 확인 후 인증되면, Access token 재발급
  7. 만약 Refresh token이 만료된 경우 재 로그인해야함

- - -

참고자료
=====

- [토큰을 저장하면 좋은 곳: Web Storage vs Cookies](https://lazyhoneyant.tistory.com/7)
- <https://tansfil.tistory.com/58?category=255594>
- <https://strongstar.tistory.com/40>
