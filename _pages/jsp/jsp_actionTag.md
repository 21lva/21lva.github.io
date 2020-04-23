---
layout: post
title:  "Action Tag"
cover:  "/assets/instacode.png"
name: java_actionTag.md
---


액션 태그
=============

- page끼리의 관계 및 이동을 제어
- 종류
  - jsp:include : 다른 페이지의 실행결과를 현재 페이지에 포함
  - jsp:forward : 페이지 이동
  - jsp:useBean : 자바빈을 사용

jsp:include
-------------------

- **<%@include >** : 디렉티브 와 달리 단순 소스 코드가 아닌 그 코드의 실행 결과를 포함
  - 디렉티브는 정적인 코드 삽입이라고 할 수 있고, 액션태그는 모듈화라고 할 수 있다
- HTML, JSP, Servlet 가능
- 이 태그를 만나면 잠시 해당 page로 이동하였다가 돌아온다
  1. jsp:include를 만나면 그 전까지의 내용을 버퍼에 저장
  2. 해당 page로 이동.
  3. page의 출력 내용을 버퍼에 저장
  4. 원래 page로 돌아와서 끝까지 진행 후 버퍼를 응답데이터로 전송

```html
<jsp:include page="<%pageName%>" flush="false">
  <jsp:param name="name" value="<%=name%>" />
</jsp:include>
```

- page : 포함할 페이지
- flush : 포함할 페이지로 가기 전에 버퍼를 flush할 것인지를 결정
  - flush="true" : 버퍼의 내용을 클라이언트에 보내고 진입. 헤더정보도 같이 보내기 때문에 이후의 내용에 그 정보를 반영할 수 없음
- jsp:param : name을 이름으로 가지는 패러미터를 같이 전송

jsp:forward
------------

- 다른 페이지로 이동
- forward 태그를 만나면 버퍼를 지우고(flush아님) 지정한 페이지로 이동

```html
<jsp:forward page="abc.jsp">
  <jsp:param name="paramName1" value="var1" />
  <jsp:param name="paramName2" value="var2" />
</jsp:forward>
```
