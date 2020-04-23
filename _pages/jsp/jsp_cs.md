---
layout: post
title:  "Cookie, Session"
cover:  "/assets/instacode.png"
name: jsp_cs.md
---

- HTTP는 응답이 끊나고 바로 연결 해제(stateless)
- 클라이언트에 연결에 대한 정보를 남김 : cookie
- 서버에 연결에 대한 정보를 남김 : session

Cookie
=======

- JSP에서 쿠키를 사용하려면 **javax.servlet.http** 의 **Cookie** 클래스의 객체를 생성해야 함
- 클라이언트의 쿠키는 사용자가 웹에 요청을 보낼때 **request** 객체에 실려서 보내짐

- 메서드
  - String getCommnet(): 쿠키에 대한 설명 반환
  - String getDomain(): 쿠키 유효한 도메인 정보 반환
  - int getMaxAge(): 쿠키의 유효 기간 반환
  - String getName(): 쿠키의 이름 반환
  - String getPath(): 쿠키의 유효한 디렉토리 정보 반환
  - boolean getSecure(): 쿠키의 보안이 설정 여부 반환
  - String getValue(): 쿠키에 설정된 값 반환
  - int getVersion(): 쿠키의 버전을 반환
  - void setComment(String): 쿠키에 대한 설명을 설정
  - void setDomain(String): 쿠키에 유효한 도메인을 설정
  - void setMaxAge(int): 쿠키의 유효한 기간을 설정
  - void setPath(Striong): 쿠키의 유효한 디렉토리를 설정
  - void setSecure(boolean): 쿠키의 보안을 설
  - void setValue(String): 쿠키의 값을 설정
  - void setVersion(int): 쿠키의 버전을 설정


생성
-----------

```java
Cookie info = new Cookie("name","value");
info.setMaxAge(365*24*60*60); //유효기간 설정. 초단위
info.setPath("/") //유효 경로 설정

response.addCookie(info);
```

사용
-------

```java
//쿠키는 여러개가 될 수 있으므로
//배열의 형태로 얻어짐
Cookie[] cookies = request.getCookies();

for(Cookie cookie : cookies){
  if(!cookie)continue;
  out.println("이름 : "+cookie.getName());
  out.println("값 : "+cookie.getValue());
}
```

삭제
----

```java
Cookie[] cookies = request.getCookies();

for(Cookie cookie : cookies){
  cookie.setMaxAge(0);//유효시간을 0으로 만들어서 만료 시킨다
  response.addCookie(cookie);//쿠키를 응답에 추가(=수정)한다
}
```

#### 로그인 예제

```html
<body>
  <form action="login" method="post">
    ID: <input type="text" name="id">  
    PASSWORD : <input type="password" name="passwd">
  </form>
</body>
```

```java
//login.java
import javax.servlet.http.Cookie;

@WebServlet("/login")
public class login extends HttpServlet{
  protected void doPost(HttpServletRequest req,HttpServletResponse res){
    doGet(req,res);
  }
  protected void doGet(HttpServletRequest req,HttpServletResponse res){
    PrintWrite out = res.getWriter();

    String id = req.getParameter("id");
    String passwd = req.getParameter("passwd");
    Cookie[] cookies = request.getCookies();
    Cookie idCookie = null;
    for(Cookie cookie : cookies){
      if(!cookie)continue;
      if(cookie.getName().equals("id")){
        idCookie=cookie;
        break;
      }
    }
    if(!idCookie)idCookie=new Cookie("id");
    idCookie.setMaxAge(60*60);

    res.addCookie(idCookie);

    res.sendRedirect("loginOk.jsp");
  }
}
```

- - -


Session
========


- 서버에 저장되는 사용자 별로 관리되는 정보. 일정 시간 후 삭제된다.
- 웹브라우저 마다 하나씩 생성되서 웹컨테이너에 저장됨
- 메서드
  - getId(): 각 접속에 대한 세션 고유의 ID를 반환(문자열)
  - getCreatingTime():
  - getLastAccessedTime():
  - getMaxInactiveInterval(): 세션의 유효시간을 초로 반환
  - setMaxInactiveInterval(t): 세션의 유효시간을 t에 설정된 초 값으로 설정
  - invalidate(): 현재 세션을 종료한다. 세션과 관련된 값들을 모두 지워진다.
  - getAttribute(name): attr로 설정된 세션값을 객체로 반환. Object로 return하기 때문에 형변환 필요
  - setAttribute(name,value): name을 key로 하고 값으로 value을 가지는 세션 설정

#### 예제

- 쓰레기 예제

```html
<body>
  <form action="login" method="post">
    ID: <input type="text" name="id">  
    PASSWORD : <input type="password" name="passwd">
  </form>
</body>
```

```java
//login.java
import javax.servlet.http.HttpSession;

@WebServlet("/login")
public class login extends HttpServlet{
  protected void doPost(HttpServletRequest req,HttpServletResponse res){
    doGet(req,res);
  }
  protected void doGet(HttpServletRequest req,HttpServletResponse res){
    PrintWrite out = res.getWriter();

    String id = req.getParameter("id");
    String passwd = req.getParameter("passwd");
    HttpSession session = request.getSession();
    if(!session.getAttribute("id")){
      session.setAttribute("id",id);
      session.setMaxInactiveInterval(30*60);
    }
    res.sendRedirect("loginOk.jsp");
  }
}
```
