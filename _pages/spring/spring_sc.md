---
layout: post
title:  "Session & Cookie"
name: spring_sc.md
---

Session & Cookie
=================

- JSP,Servlet에서 다루던 것과 비슷하게 사용
- client와 server의 연결을 유지해주는 데에 사용

Session
========

- spring에서 **HttpServletRequest** 를 이용해서 session을 사용하고자 하면 Controller에 **HttpServletRequest** 를 매개변수로 받아야 한다

```java
@RequestMapping(value="/login",method=RequestMethod.POST)
public String userLogin(Model model,HttpServletRequest request)
  //생략
  HttpSession session = request.getSession();
  session.setAttribute("isLogined",true);
  //생략
  //수정
  boolean newS = !session.getAttribute("isLogined");
  session.setAttribute("isLogined",newS);
  //삭제
  session.invalidate();
}
```

- HttpServletRequest가 아닌 매개변수로 **HttpSession** 을 받아서 사용 가능
- 차이는 객체를 얻는 방법일 뿐. 다른 부분에서는 거의 차이가 없다

```java
@RequestMapping(value="/login",method=RequestMethod.POST)
public String userLogin(Model model,HttpSession session)
  //생략
  session.setAttribute("isLogined",true);
  //생략
}
```

todo : 메서드 자세히 from jsp
- 메서드
  - getId() : session id 반환
  - setAttribute("이름",value) : 속성 지정
  - getAttribute() : 속성 반환
  - removeAttribute() : 속성 제거
  - setMaxInactiveInterval() : 객체의 유지 시간 설정
  - getMaxInactiveInterval() : 객체의 유지 시간 반환
  - invalidate() : 세션 객체 제거

Cookie
--------

- **HttpServletResponse** 객체에 저장해서 클라이언트에 보냄
- todo : cookie

#### @CookieValue

```
public string f1(Model model,@CookieValue(value="id",required=false) Cookie idCookie, HttpServletRequest request){
  if(idCookie)model.setAttribute("id",idCookie.getValue());
  //생략
}
```
