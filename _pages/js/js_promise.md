---
layout: post
title:  "Promise"
date:   2019-05-17 01:02:59
author: Inhyuk
categories: Javascript
category: Javascript
tags: js
cover:  "/assets/instacode.png"
name: js_promise.md
---

Promise
========

Callback Hell
------------

- 함수를 순차적으로 실행하고자 하면 callback을 지속적으로 쓰게 되고 **Callback Hell** 에 빠질 수 있다.

```js
func1(
  callback1(
    callback2(
      callback3(
        callback4(
          callback5();
          );
        );
      );
  );
)
```

Promise
-------

- **Promise** 는 비동기적 실행을 비동기 처리에 사용되는 객체
- [Micro Task](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)의 글을 읽어보면 Promise의 동작방식을 어느정도 알 수 있다.

선언부
-----

```js
let f2 = function(args){
  return new Promise((resolve,reject)=>{
    //실행
    if(fullfilled)resolve();
    else reject();
  });
}
let f1 = new Promise(function (resolve, reject) {
  //실행부
  });
```

- 프라미스는 3가지 상태가 존재. 객체가 생성되면서 작업을 한다
1. pending : 아직 수행을 하지 않은 상태. 프라미스 객체를 생성할 때의 상태
2. fullfilled : 내부 작업이 성공한 상태. **resolve** 가 수행되고 값을 return 한다. **then** 메서드를 수행할 수 있다
3. rejected : 작업이 실패한 상태, **reject** 를 이행되고 값을 return 한다. 전달해준 값은 **catch** 메서드에서 사용

```js
var getData = function(){
  return new Promise(function (resolve, reject) {
    $.get("/apc/1",(response)=>{
      if(response){
        resolve();
      }
      else{
        reject();
      }
      })
  });
};

//사용
getData().then(res,rej);//메서드를 넣어주도록 하자. 메서드를 콜한 것을 넣어주면 안됨

//catch를 사용하는 방법
getData().then(res).catch(rej);
```

체이닝
------

- *then* 메서드를 호출하면 새로운 *Promise* 객체를 리턴한다. return을 통해서 값을 전달한다(또 다른 then 메서드를 사용하면)

```js
var promise = new Promise(res,rej);

promise.then(calback1(){

  }).then(callback2(){

    });

//또는
Promise.resolve().then(callback1(){

  }).then(callback2(){

    });

//이전 callback의 return 값을 사용 가능
Promise.resolve().then(()=>{
  console.log("Let's return 2");
  return 2;
  }).then((data)=>{
    console.log(data);
    });
```

- 다음과 같이 promise를 연결할 수도 있다

```js
function firstPromise(data){
  return new Promise((resolve,reject)=>{
    console.log(data);
    resolve(data);
  })
}
function secondPromise(data) {
  return new Promise((resolve,reject)=>{
    console.log(data+1);
    resolve(data+1);
  });
}

function thirdPromise(data) {
  return new Promise((resolve,reject)=>{
    console.log(data+1);
    resolve(data+1);
  });
}
function lastPromise(data) {
  return new Promise((resolve,reject)=>{
    console.log(data+1);
    resolve(data+1);
  });
}

firstPromise(1)
  .then(secondPromise)
  .then(thirdPromise)
  .then(lastPromise);
//[result]
//1
//2
//3
//4
```

- Promise가 이행하거나 거부했을 때, 각각에 해당하는 핸들러 함수(onFulfilled나 onRejected)가 비동기적으로 실행됩니다. 핸들러 함수는 다음 규칙을 따라 실행됩니다.[출처](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)
  1. 함수가 값을 반환할 경우, then에서 반환한 프로미스는 그 반환값을 자신의 결과값으로 하여 이행합니다.
  2. 값을 반환하지 않을 경우, then에서 반환한 프로미스는 undefined를 결과값으로 하여 이행합니다.
  3. 오류가 발생할 경우, then에서 반환한 프로미스는 그 오류를 자신의 결과값으로 하여 거부합니다.
  4. 이미 이행한 프로미스를 반환할 경우, then에서 반환한 프로미스는 그 프로미스의 결과값을 자신의 결과값으로 하여 이행합니다.
  5. 이미 거부한 프로미스를 반환할 경우, then에서 반환한 프로미스는 그 프로미스의 결과값을 자신의 결과값으로 하여 거부합니다.
  6. 대기 중인 프로미스를 반환할 경우, then에서 반환한 프로미스는 그 프로미스의 이행 여부와 결과값을 따릅니다.


Promise.all & Promise.race
--------

- **Promise.all** : Promise 객체 배열을 전달 받고 모든 객체의 상태가 **fullfiled** 가 되면(모든 객체 동시에 실행), **then** 메서드를 호출한다.
- **Promise.race** : Promise 객체 배열을 전달 받고 상태가 **fullfiled** 가 되는(모든 객체 동시에 실행) 객체가 생기면, **then** 메서드를 호출한다.

- - -

async & await
=============

- ES7에 추가 됨
- 아래는 보통의 async, await 함수 모양이다
- async 함수는 Promise 객체를 리턴한다

```js
async function f1() {
  await asyncFunction();
}
```

- await의 대상이 되는 함수는 Promise 객체를 반환하는 함수이다.
- await는 Promise 객체가 **fullfiled** 가 될 때까지 기다리고, **resolve** 나 **return** 으로 전달되는 값(이행된 결과 값)을 저장한다

```js
function forAwait(){
  return new Promise((resolve)=>{
    console.log("Promise");
    resolve(11);
  });
}

async function f1(){
  var resultVal = await forAwait();
  console.log(resultVal);
}

function Fetch(url) {
  return fetch(url).then(function(response) {
    return response.json();
  });
}

async function fff() {  
  var user = await Fetch("www.naver.com");
  console.log(user);
}
```

- 비동기적인 함수를 통해서 배열의 처리를 하고자 하면 다음과 같이 한다([출처](https://mygumi.tistory.com/328))

```js
function delay(item) {
  return new Promise(resolve =>
    setTimeout(() => {
      console.log(item);
      resolve();
    }, 500)
  );
}

async function serial(array) {
  //순차적으로 하나씩 수행
  for (const item of array) {
    await delay(item);
  }
  console.log("Done!");
}


async function parallel(array) {
  //병렬적으로 수행
  const promises = array.map(item => delay(item));
  await Promise.all(promises);
  console.log("Done!");
}
```

- 예외 처리는 **try-catch** 를 통해서 가능하다

```js
async function fff() {
  try {
    var data = await asyncFunction();
  } catch (error) {
    console.log(error);
  }
}
```

- - -

참고 자료
========

- [Promise1](https://velog.io/@cyranocoding/2019-08-02-1808-%EC%9E%91%EC%84%B1%EB%90%A8-5hjytwqpqj)
- [Promise2](https://joshua1988.github.io/web-development/javascript/promise-for-beginners/)
- [Promise.all](https://medium.com/sjk5766/%EB%92%A4%EB%8A%A6%EA%B2%8C-%EC%A0%95%EB%A6%AC%ED%95%98%EB%8A%94-promise-9dc0592cdc34)
- [async,await](https://mygumi.tistory.com/328)
- [async,await](https://www.daleseo.com/js-async-async-await/)
- [promise, async,await의 관계](https://medium.com/@kiwanjung/%EB%B2%88%EC%97%AD-async-await-%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-%EC%A0%84%EC%97%90-promise%EB%A5%BC-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-955dbac2c4a4)
