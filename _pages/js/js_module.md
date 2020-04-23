---
layout: post
title:  "모듈"
date:   2019-05-17 01:02:59
author: Inhyuk
categories: Javascript
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_module.md
---

모듈
===

- 자바스크립트 자체에 모듈은 존재하지 않음
- 호스트 환경에 따라 서로 다른 모듈화 방법 제공
- 자주 사용하는 코드를 별도의 파일로 만들거나 비슷한 기능을 하는 코드들을 묶어서 만듬
- 필요한 로직만 로드해서 메모리 낭비 방지
- 코드 수정이 용이
- 한 번 다운로드 된 모듈은 웹브라우저 자체에 저장해서 로드 시간과 트래픽을 절약함
- 호스트 환경 : 자바스크립트가 구동되는 환경

생성
----

- *[파일이름].js* 로 생성

export, exports, export default
-------------------------------

- export : 모듈에서 함수, 객체, 원시값을 내보낼 때 사용
- exports : 모듈에서 객체의 형태로 내보낼 때 사용
- exports default : 하나의 고정된 값만 내보낼 때 사용. [exports vs module.exports](https://dydals5678.tistory.com/97)

#### export

```js
//모듈 파일
export const a=1;
export function hello(){
  console.log("HI");
}

//require 로 이용
var moduleA = require("./a.js");
moduleFile.a;
moduleFile.hello();

//import로 이용
import {a,hello} from "./a.js";
console.log(a);
hello();
```

#### exports

```js
//모듈 파일
exports.a=1;
exports.hello=function(){
  console.log("HI");
}

//require 로 이용
var moduleA = require("./a.js");
moduleFile.a;
moduleFile.hello();

//import로 이용
import moduleA from "./a.js";
moduleFile.a;
moduleFile.hello();
```

#### export default

```js
//모듈 파일
export default function () { ... };

//require 로 이용
var moduleA = require("./a.js");
moduleA();

//import로 이용
import moduleA from './a.js';
moduleA();
```


- [설명1](https://velog.io/@jch9537/Javascript-export-default)
- [설명2](https://medium.com/@enro2414_40667/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-export-import%EC%A0%95%EB%A6%AC-137ac9e327d9)

- - -

라이브러리
========

- 모듈들의 집합이라고 보면 됨.
- *jQuery* : 가장 많이 사용되는 라이브러리.
