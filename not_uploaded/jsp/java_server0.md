---
layout: post
title:  ""
cover:  "/assets/instacode.png"
name: java_server0.md
category: JSP
---

웹서버 vs 웹어플리케이션 서버
=====

- [참고자료](https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html)

웹서버
----

- 클라이언트로부터 HTTP 요청을 받아들이고, HTML문서과 같은 정적 컨텐츠를 반환해주는 프로그램
- 동적인 컨텐츠 제공을 위한 요청을 전달하고 응답을 받음
- 인증, 정적 컨텐츠 관리 등을 수행
- 종류
  - apache
  - nginx

웹 어플리케이션 서버(WAS)
------------------

- HTTP를 통햇 컴퓨터 장치에 어플리케이션을 수행하는 미들웨어
- 동적 서버 컨텐츠를 수행
- 트랜잭션을 관리하고, 데이터베이스 접속 기능을 제공
- 현재 WAS들은 웹서버의 정적인 컨텐츠 기능도 가지고 있음
- 종류
  - Tomcat
  - JBoss

WAS + Web Server가 필요한 이유
---------------------------

- 기능을 분리하여, 단순 정적 컨텐츠를 빠르게 웹서버를 통해서 제공
- 정적 컨텐츠까지 WAS하기에는 부하가 커짐
- 물리적으로 분리하여 보안강화
- 여러대의 WAS와 연결. Load Balancer을 이용하여 부하를 분산시킬 수 있음
- 하나의 서버에 여러 개의 다양한 WAS 이용 가능
- 웹서버를 WAS 앞에 두고 필요한 WAS들을 웹서버의 플러그인 형태로 설정하면 효율적

CGI
----

- 웹서버에서 어플리케이션을 동작시키기위한 인터페이스
- 초기 웹 프로그램에서 사용
- 프로세스 방식으로 서버의 부하가 심함
- 같은 기능을 수행하더라도 처음부터 메모리에 로드해야 함 -> 메모리에 과부하

Sevlet
--------

- Tomcat이 이해할 수 있는 순수 Java코드로 이루어진 웹서버용 클래스
- 스레드 단위로 실행
- 자바안에 HTML코드가 포함되어 있음
- print() 등이 불편함
- MVC의 Controller에 많이 사용


JSP
---

- html안에 자바 코드가 포함된 서버사이드 스크립트
- sevlet과 달리 html표준에 맞게 작성됨
- 스레드 방식으로 실행
- 클라이언트 요구를 처리하는 기능은 최초 한번만 메모리에 로드(재사용)
- MVC의 view에 많이 사용

- - -

톰캣
======

- Tomcat은 HTTP(HyperText Transfer Protocol) Servlet container
- Servlet을 실행하고 JSP, JSF를 Servlet으로 변환
- Java EE Container 의 기능을 포함한 WAS


톰캣의 컨테이너 구조
------------------


- 웹어플리케이션
  - WEB-INF : 웹 어플리케이션에 관한 정보를 저장한 디렉토리. 외부에서 접근 불가능
    - classes : java class 파일들 + servlet
    - lib : jar 라이브러리
    - web.xml : 환경설정 파일. 각종 servlet 설정 보유
  - META-INF : context.xml 보유
  - jsp/html : JSP와 HTML 파일이 저장되는 디렉토리
  - css : 스타일 시트가 저장되는 곳
  - image : 이미지가 저장되는 곳
  - js : 자바스크립트 파일이 저장되는 곳
  - bin : 어플리케이션에서 사용되는 각종 실행 파일이 저장되는 디렉토리
  - conf : 프레임워크에서 사용하는 각종 설정 파일이 저장되는 디렉토리
  - src : 자바 소스 파일이 저장되는 디렉토리

컨테이너에서 웹 어플리케이션 실행
------------------------------

- 컨텍스트 : server.xml에 등록하는 웹 어플리케이션
- 웹어플리케이션당 하나의 컨텍스트 등록
- 웹어플리케이션과 이름이 다를 수도 있지만, 중복은 안됨

- **Context** 태그의 구성 요소. Host 태그 안에 설정
  - path : 웹 어플리케이션의 이름. 브라우저에서 웹 어플리케이션을 요청하는 이름
  - docBase : 컨텍스트에 대한 실제 웹 어플리케이션이 위치한 경로. WEB-INF의 상위 디렉토리 까지의 경로
  - realoadable : 실행 중 소스코드가 수정될 경우 바로 갱신될 지 여부. false=다시 실행해야 수정된 부분 반영

```html
<!--
코드 부분
-->

<Context path="/webMall" docBase="C:\webShop" reloadable="true"/>


<!--
코드 부분
-->
```
