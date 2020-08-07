---
layout: post
title:  "Servlet"
cover:  "/assets/instacode.png"
name: java_servlet.md
category: JSP
---

웹컨테이너 구조
==============

- xxx.jsp -(request)-> **xxx.jsp.java -> xxx_jsp.class -> xxx_jsp.obj** -(reponse)-> HTML
- 웹 컨테이너 : WAS의 구성요소. 개발자가 만든 jsp파일을 위의 과정을 통해서 작업해서 response한다

Servlet
========

- 순수한 자바파일로 웹페이지를 구성하는 것

servlet mapping
----------------

- URL mapping : 어떤 url을 통해서 해당 servlet을 실행해줘야 하는지를 알려줌
- full path : url/port/contextPath/servelt폴더이름/servlet이름
  - 보안에 취약
- mapping path : url/port/contextPath/servletMapping이름

#### 1. web.xml 파일을 이용

```xml
<!--
Code
-->
<servlet>
  <servlet-name>servletEx</servlet-name>
  <servlet-class>패키지네임을 포함한 클래스</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>servletEx</servlet-name>
  <url-pattern>매핑 주소</url-pattern>
</servlet-mapping>
<!--
Code
-->
```

```java
package com.hello;

import javax.servlet.http.*;
import java.io.*;

@SuppressWarnings("serial")
public class hello extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");
        request.setCharacterEncoding("utf8");
        response.setCharacterEncoding("utf8");
        PrintWriter out = response.getWriter();
        out.println("<html><head></head><body>");
        out.println("<h1>Hello, world</h1><p>this is sample servlet.</p>");
        out.println("</body></html>");
    }
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
    }
}
```

#### 2) Annotation 사용

```java
package com.hello;

import javax.servlet.http.*;
import java.io.*;

@SuppressWarnings("serial")
@WebServlet("/hl")
public class hello extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");
        request.setCharacterEncoding("utf8");
        response.setCharacterEncoding("utf8");
        PrintWriter out = response.getWriter();
        out.println("<html><head></head><body>");
        out.println("<h1>Hello, world</h1><p>this is sample servlet.</p>");
        out.println("</body></html>");
    }
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
    }
}
```

- - -

Servlet Request, Response
========================

HttpServlet
----------

- 개발자가 servlet 객체를 만들기 위해서 상속 받는 **추상 클래스**

HttpServletRequest
-----------------

- client로부터 오는 요청을 담은 클레스
- 메서드
  - getCookies()
  - getSession()
  - getAttribute()
  - setAttribute()
  - getParameter()
  - getParameterName()
  - getParameterValues()

HttpServletResponse
-------------------

- 서버로부터 보내는 응답을 담은 클레스
- 메서드
  - addCookies()
  - getStatus()
  - sendRedirect()
  - getWriter()
  - getOutputStream()

- - -


Servlet Lifecycle
========

- PostConstruct -> init -> service -> destroy -> PreDestroy
  - PostConstruct : servlet 객체 생성 직후 호출되는 함수
  - init : 최초 한번 호출되는 함수.
  - service : 잘 안쓰임. doGet, doPost 등을 많이 씀
  - destroy : 서블릿 마무리 단계에 한번 웹컨테이너에 의해 자동으로 호출됨
  - PreDestroy : destroy()이후 호출

- 최초 요청시 객체를 생성. 이 후에는 객체는 컨테이너에서 대기

```java
import java.io.IOException;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/LifeCycleServlet")
public class LifeCycleServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public LifeCycleServlet() {
        super();
    }

    @PostConstruct
    public void postConstructMethod(){
        System.out.println("PostConstruct 호출");
    }

    @PreDestroy
    public void preDestroyMethod(){
        System.out.println("PreDestroy 호출");
    }

    @Override
    public void init() throws ServletException {
        super.init();
        System.out.println("init() 호출");
    }

    @Override
    public void destroy() {
        super.destroy();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
}
```

ServletConfig
---------------

- DD(deployment descriptor) : WEB-INF 아래 web.xml로 존재. 서버 시작시 메모리에 로딩됨. 다양한 환경 정보를 보유
- web.xml에 config값을 적고, ServletConfig 객체를 만들어서 사용(한 servlet당 하나씩)
- read only

- web.xml의 모습

```xml
<servlet>
  <servlet-name> 서블릿 객체 이름 </servlet-name>
  <servlet-class> 서블릿 클래스 이름(패키지 명을 포함한 풀 네임) </servlet-class>
  <init-param>
    <param-name> 초기파라미터 이름 </param-name>
    <param-value> 초기 파라미터 값 </param-value>
  </init-param>
</servlet>
```

- servlet의 모습

```java
@WebServlet("/myServlet")
public class MyServlet extends HttpServlet{

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletConfig config = getServletConfig();
        String id = config.getInitParameter("id");
        String passwd = config.getInitParameter("passwd");

        System.out.println(id);
        System.out.println(passwd);
    }

}
```

ServletContext
-----------------

- 서블릿끼리 자원을 공유
- read, write 가능

```xml
<context-param>
  <param-name> 초기파라미터 이름 </param-name>
  <param-value> 초기파라미터 값 </param-value>
</context-param>
```

```java
ServletContext context = getServletContext();
String fileName = context.getInitParamter("name");
```

- - -

form
====

```java
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Arrays;
import java.util.Enumeration;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/FormEx")
public class FormEx extends HttpServlet {
    private static final long serialVersionUID = 1L;

    //생성자
    public FormEx() {
        super();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("doGet");
        String id = request.getParameter("id");
        String pw = request.getParameter("pw");

    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        System.out.println("doPost");
        request.setCharacterEncoding("utf-8");
        //return타입은 String
        String id = request.getParameter("id");
        String pw = request.getParameter("pw");

        //array 형태(ex. checkbox)
        String[] hobbys = request.getParameterValues("hobby");

        // request.getParameterNames() : 넘긴 name값을 Enumeration 타입으로 반환함
        Enumeration<String> values = request.getParameterNames();
        while(values.hasMoreElements()){
            String param = values.nextElement();
            System.out.println(request.getParameter(param));
        }

        // 응답 문서 인코딩 타입 지정
        response.setContentType("text/html;charset=utf-8");

        // 문서 출력 스트림 객체 얻기
        PrintWriter writer = response.getWriter();

        writer.println("<head></head>");
        writer.println("아이디 : " + id + "<br/>");
        writer.println("비밀번호 : " + pw + "<br/>");
        writer.println("취미 : " + Arrays.toString(hobbys) + "<br/>");
        writer.println("");
        // 자원 해제
        writer.close();

    }
  }
```

- - -

참고자료
=======

- <https://rongscodinghistory.tistory.com/58?category=705081>
