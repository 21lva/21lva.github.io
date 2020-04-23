---
layout: post
title:  "배열"
date:   2019-05-15 01:02:59
author: Inhyuk
categories: Javascript
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_array.md
---


배열
===

1) 선언
---

- 객체의 형태
- c와 달리 배열의 요소들의 타입이 달라도 됨.
- 빈 배열 선언 가능

```js
<script>
let arr = [1,2,3,"hi"];
for(let i =0;i<arr.length; ++i){
  document.write(arr[i]);
}
document.write("<br>"+typeof(arr));
</script>
```



2) 메소드
---

- push : 배열의 끝에 원하는 값 추가
- pop : 배열의 마지막 원소 제거
- shift : 배열의 맨 처음 요소 제거 후 그 요소 반환
- length : 배열의 길이
- concat : 두 배열을 합친 배열을 반환(두 배열은 변화 없음)
- join : 특정 문자열을 중간중간에 삽입하여 배열의 원소들로 문자열 생성
- sort/reverse : 정렬/역순정렬
- slice(start,end) : 배열의 일부( [start,end) )를 반환

```js
<script>
let arr = [1,2,3];
for(let i =0;i<arr.length; ++i){
  document.write(arr[i]);
}
document.write("<br>"+typeof(arr));
</script>
```


- - -

참고자료
======

- [프로토타입 설명](https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67)
- [모듈패턴](https://poiemaweb.com/js-object-oriented-programming)
