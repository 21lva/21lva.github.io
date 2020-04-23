---
layout: post
title:  "Redirect & Interceptor"
name: spring_ri.md
---

Redirect & Interceptor
===================

- redirect : controller 내에서 분기
- intercept : controller 실행 전 or 후로 작업

Redirect
---------

- 특정 페이지로 전환하는 기능

```java
//index 페이지로 redirect
return "redirect:/";

//ModelAndView 사용
mav.setViewName("redirect:/");
```

Interceptor
------------

- redirect가 빈번하게 되는 경우 **HandlerInterceptor** 를 이용
- Controller로 보내기 전에 **HandlerInterceptor** 가 먼저 호출됨
- 3가지 메서드 사용. 반환 값이 true이면 controller로 넘어간다
  - preHandle(): Controller가 작동하기 전. 자주 쓰임. redirect 대체 가능
  - postHandle(): Controller가 작동한 후
  - afterCompletion(): Controller와 View가 작동한 후

- HandlerInterceptor가 interface이기 때문에 **HandlerInterceptorAdaptor** 클래스를 상속받아서 사용

```java
public class myInterceptor extends HandlerInterceptorAdaptor{
  @Override
  public boolean preHandle(HttpServletRequest request,HttpServletResponse reponse){
    HttpSession session = request.getSession();
    if(!session){
      //생략
      return true;
    }
    response.sendRedirect("/");
    return false;
  }
}
```

- context.xml을 수정해서 만든 Interceptor가 적용될 controller들을 지정할 수 있다

```xml
<interceptors>
  <interceptor>
    <mapping path="적용될 class경로"/>
    <mapping path="적용될 class경로"/>
    <exclude-mapping path="적용될 class경로"/><!--적용하지 않을 클래스 지정. 주로 범위로 지정한 곳에서 특정 클래스를 제외할 때 사용-->
    <beans:bean class="패키지/interceptor클래스"/>
  </interceptor>
</interceptors>
```
