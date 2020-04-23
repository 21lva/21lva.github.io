---
layout: post
title:  "jQuery 3 - 비동기 처리 & 플러그인"
date:   2019-06-13 02:02:59
author: Inhyuk
categories: Javascript
tags: js
cover:  "/assets/instacode.png"
name: jquery_async.md
---

Ajax
=====

- Ajax : 비동기 방식의 javascript와 XML. 서버에 자료를 요청할 때 화면 전환 없이 요청한 자료를 전송 받을 수 있음. (ex. 로그인할 시에 페이지 전체 로드 없이 로그인 여부만 확인)
- 동기 방식 : 서버에 신호를 보내고 응답이 돌아와야 다음 동작을 수행
- Action page : 서버 측에 저장되어 있어서 전송된 데이터를 확인한 후 요청한 응답을 전송해줄 페이지. 서버 측 스크립트(PHP,JSP,ASP 등)사용.

- - -

XMLHttpRequest
==============

- XMLHttpRequest(XHR) : 서버측과 ajax를 하기 위해 사용되는 객체
- open(방식, url, 동기=false or 비동기=true) : 초기화, get/post 선택. url 입력
- send() : 서버에 요청 전송. GET일 경우 null을 넣어준다. POSt의 경우에는 서버로 보내고 싶은 데이터를 입력한다.
- onreadychange : 서버로부터 응답이 도착하면 호출될 함수를 가지는 XHR객체의 프로퍼티
- readyState : 현재의 XHR상태이다. 콜백함수는 실제로 응답을 받을때가 아닌 readyState가 4가 될 때 호출된다.
1. 0 : uninitialized. 객체만 생성되고 초기화되지 않은 상태(open 실행 전)
2. 1 : loading. open과 send사이
3. 2 : loaded. send를 한 후 데이터를 받지 않은 상태(status와 헤더가 도착하지 않음)
4. 3 : interactive. 데이터의 일부를 받은 상태
5. 4 : completed. 데이터를 전부 받은 상태

- status(statusText): 서버로부터의 응답의 상태
1. 200(OK) : 요청 성공
2. 403(Forbidden) : 접근 거부
3. 404(Not Found) : 페이지 없음
4. 500(Internal Server Error) : 서버 오류

- reponseText : 서버로부터 받은 텍스트. readyState가 4이고 status가 200일 때 사용.

```html
<script>
  //XHR 객체 생성
  var httpRequest=null;
  var getXHR = function(){
    if(window.XMLHttpRequest) return new XMLHttpRequest();
    else return null;
  }
  httpRequest = getXHR();

  //get 방식으로 전송
  httpRequest.open("GET","/test.jsp?id=inhyuk&pw=1111",true);
  httpRequest.send(null);

  //post 방식으로 전송
  httpRequest.open("POST","/test.jsp",true);
  httpRequest.send("id=inhyuk&pw=1111");

  //응답시 호출된 callback 함수 정의
  var callbackFunction=funtion(){}

  //서버로부터 응답이 도착하면 호출될 callback 함수를 XHR의 프로퍼티로 지정
  httpRequest.onreadystatechange=callbackFunction;
</script>
```

- - -

jQuery + Ajax
==============

1) load()
-----------

```js
$("요소선택").load(url,data,콜백함수)
//데이터, 콜백함수 생략 가능
```

- 사용자가 지정한 URL 주소에 데이터를 전송하고, 외부 콘텐츠를 요청한다.

```html
<!--ajax.html 일부-->
<body>
<p id="P1">hello</p>
</body>

<!--index.html 일부-->
<input type="button" value="Push" onclick="
$('#pa1').load('ajax.html #P1');
">

<div id="pa1"></div>
```

2) ajax()
--------------

- 지정한 URL 경로에 파일의 데이터를 전송하고 입력한 URL 경로 파일로부터 요청한 데이터를 불러온다(text,html,xml,json 등의 형식)

```js
$.ajax({
  url:"전송 페이지",
  type:"전송방식(get,post)",
  data: "전송데이터",
  dataType:"요청한 데이터 형식(html,xml,json,text)",
  success:function(data){
    전송에 성공하면 실행될 코드
  },
  error:function(){
    전송에 실패하면 실행될 코드
  }
  })
```

- 다음과 같은 옵션도 적용할 수 있다.
1. async : 비동기(true,기본), 동기(false) 방식으로 설정
2. beforeSend : 요청하기 전에 함수를 실행하는 이벤트 핸들러
3. cache : 요청한 페이지를 인터넷의 캐시에 저장할지 여부. 기본값은 true
4. complete : Ajax가 완료되었을 때 함수를 실행하는 이벤트 핸들러
5. timeout : 통신시간을 제한한다. 밀리 second단위
6. username : HTTP 엑세스를 할 때 인증이 필요한 경우에 쓰이는 사용자 이름

- 비동기 방식으로 데이터를 전송하려면 데이터를 가공해야 한다.

```js
//serialize() : 데이터의 전송방식을 쿼리 스트링 형식의 데이터로 변환
$("from").serialize()

//serializeArray() : 배열 객체로 변환해 반환
$("from").serializeArray()

//param() : {name1:value1 , name2:value2} -> "name1=value1 & name2=value2"
$.param(Object);
```
- ajax 예제. 버튼을 누르면 아래의 숫자만 바뀐다

```html
<body>
    <p id = "counter"></p>
    <button onclick="cl()"></button>
    <p id = "log">123</p>
    <script>
        var x=1;
        let start = new Date().getTime();
        $('#counter').text(start);
    function cl(){
        $.ajax({
            url:'/board/ajax',
            dataType:'json',
            type:'POST',
            success:function(result){
                let start = new Date().getTime();
                $('#log').text(start);
            },
            error:function(result){
                let start = new Date().getTime();
                $('#log').text(start);
            }
        });
    }
   </script>
</body>
```

3) Ajax로 json binding
-------------------

- binding : 비동기 통신 기술을 이용하여 서버 DB에 데이터를 요청하고 그 데이터를 받아서 HTML 태그에 결합하는 것

4) 동일 출처 정책(same-origin policy)
-------------------------------------

- 자바스크립트는 스크립트를 포함하는 문서와 같은 **출처**(불러온 URL의 프로토콜, 호스트, 포트로 정의) 의 문서에 있는 window와 Document 객체만을 이용할 수 있다.
- 즉, 다른 서버에서 불러온 문서의 내용은 이용할 수 없다.
- 서로 다른 사이트(서버)에 데이터를 주고 받으려면 서버 언어를 이용해야한다.
- RSS : 초간편 배포. 자주 갱신되는 뉴스 등에 사용되는 XML 기반의 포맷. PHP를 이용하여 서버에서 데이터를 불러와서 사용하기도 한다.

- - -

플러그인
=======

- jQuery Plug-in : 다른 개발자가 구현해 놓은 프로그램을 자바스크립트 파일로 제공하는 라이브러리(저작권 잘 살펴보자.)

추후 추가 예정

- - -

참고 자료
=========

[XHR 관련 자료](https://myeonguni.tistory.com/1526)
