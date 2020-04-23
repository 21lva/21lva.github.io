---
layout: post
title:  "JSP"
cover:  "/assets/instacode.png"
name: java_object.md
---


JSP
====

- servlet : 순수 자바 코드. MVC의 Controller
- JSP : HTML안에 자바코드를 넣음. 컨테이너가 servlet으로 만들어냄. MVC의 view

- 디렉티브(<%@    %>) : JSP에 대한 설정
  - page : 페이지에 대한 정보
  - taglib : 사용할 태그 라이브러리 지정
  - include : 특정 문서 포함
- 선언부(<%!   %>): 전역변수 or 메서드 선언
- 스크립트(<% %>): 자바 코드 작성할때 사용
- 표현식(<%=   %>) : 값을 사용

- - -

JSP 내장객체
============

- 내장객체 : JSP에서 따로 포함하거나 선언하지 않고 사용할 수 있는 객체
- servlet으로 변환될 때 멤버변수, 메서드, 매개변수 등의 참조 변수로 바뀐다
- request, response, session, page, pageContent, config, out 등이 존대

Response & Request 객체
-------------------------

- Request: 클라이언트로부터 서버로 보내는 요청의 객체
  - getParameter(String name) : parameter값 얻음
  - getParameterValues(String name) : parameter값 얻음
  - getParameterNames(String name) : name 모두 얻음
  - getCookies() : 모든 쿠키값 가져옴
  - getMethod() : 현재 요쳥의 종류(GET,POST)
  - getSession() : 세션을 가져옴

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
</head>
<body>
    <%
        out.println("서버 : " + request.getServerName() + "</br>");
        out.println("포트번호 : " + request.getServerPort() + "</br>");
        out.println("요청방식 : " + request.getMethod() + "</br>");
        out.println("프로토콜 : " + request.getProtocol() + "</br>");
        out.println("URL : " + request.getRequestURL() + "</br>");
        out.println("URI : " + request.getRequestURI() + "</br>");
    %>
</body>
</html>
```

- Response: 서버에서 클라이언트로 보내는 응답 객체
  - setContentType(type) : 문자열 형태에 맞는 content type 설정
  - setHeader(name,value)
  - sendRedirect(url): 다른 페이지로 보냄

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
    <form action="re.jsp">
        <input type="text" name="age" size="5">
        <input type="submit" value="확인">
    </form>
</body>
</html>

------------------------------------


<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    <%! int age; %>
    <%
        String str = request.getParameter("age");
        age = Integer.parseInt(str);
        if(age >= 20){
            response.sendRedirect("yes.jsp");
        }else{
            response.sendRedirect("no.jsp");
        }
    %>
</body>
</html>
```

out
-----

- 출력 스트림. 사용자의 웹 브라우저에 출력하는 데에 사용됨
- 버퍼관련 메서드 제공

- 메서드
  - getBufferSize(): output buffer의크기를 바이트로 반환
  - clearBuffer(): 버퍼에 비운다
  - flush(): 버퍼를 비우고 output stream도 비운다
  - close(): output stream을 닫고 버퍼를 비운다
  - println(content): content의 내용을 출력 후 개행
  - print(content): content의 내용을 출력

session
----------

- 서버에 저장되는 사용자 별로 관리되는 정보. 일정 시간 후 삭제된다
- 메서드
  - getId(): 각 접속에 대한 세션 고유의 ID를 반환(문자열)
  - getCreatingTime():
  - getLastAccessedTime():
  - getMaxInactiveInterval(): 세션의 유효시간을 초로 반환
  - setMaxInactiveInterval(t): 세션의 유효시간을 t에 설정된 초 값으로 설정
  - invalidate(): 현재 세션을 종료한다. 세션과 관련된 값들을 모두 지워진다.
  - getAttribute(name): attr로 설정된 세션값을 객체로 반환. Object로 return하기 때문에 형변환 필요
  - setAttribute(name,value): name을 key로 하고 값으로 value을 가지는 세션 설정

config
--------

- servlet이 초기활 될때 사용하는 **ServletConfig** 객체
- web.xml의 정보 사용
- 메서드
  - Enumeration getInitParameterNames(): 모든 초기화 파라미터 이름을 리턴
  - String getInitParameter(name): 이름이 name인 초기화 파라미터의 값을 리턴
  - String getServletName(): 서블릿의 이름을 리턴
  - ServletContext getServletContext(): 실행하는 서블릿 ServletContext 객체를 리턴

application
-------------

- 웹 어플리케이션(컨텍스트) 전체에 대한 정보를 관리
- servletContext 객체에 대한 참조변수. 컨테이너의 정보 제공
- 메서드
  - getServerInfo(): JSP/서블릿 컨테이너의 이름과 버전을 반환한다.
  - getMajorVersion(): 컨테이너가 지원하는 서블릿 API의 주 버전정보를 반환한다.
  - getMinorVersion(): 컨테이너가 지원하는 서블릿 API의 하위버전 정보를 반환한다.
  - log(message): 문자열 message의 내용을 파일에 기록한다. 로그파일의 위치는 컨테이너에 따라 다르다.
  - log(message,exception): 예외 상황에 대한 정보를 포함하여 로그 파일에 기록한다.
  - getAttribute(String name): 문자열 name에 해당하는 속성 값이 있다면 Object형태로 가져온다. 따라서 변환 값에 대한 적절한 형 변환이 필요하다.
  - getAttributeNames(): 현재  application객체에 저장된 속성들의 이름을 열거 형태로 가져온다.
  - setAttribute(String name, Object Value): 문자열 name이름으로 Object형 데이터를 저장한다. Object형이므로 자바 클래스 형태로도 저장 할 수 있다.
  - removeAttribute(String name): 문자열 name에 해당하는 속성을 삭제한다.


참고자료
========

- <https://dear-by-dear.tistory.com/21>
- <https://yoonka.tistory.com/177>
- <https://dhzzang.tistory.com/85>
- <https://hyeonstorage.tistory.com/78>
