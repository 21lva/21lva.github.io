---
layout: post
title:  "디렉토리 구조 & 동작 과정"
name: spring_structure.md
---

구조
====

```
- pom.xml
- src-main- java
         |- resources
         |- webapp- resources
                 |- WEB-INF- classes
                          |- spring- appServlet- servlet-contex.tml
                                  |- root-context.xml
                          |- views - *.jsp
                          |- web.xml
```

- src/main/java : java 파일들.Controller, Service, DAO 등
- src/main/webapp : 웹과 관련된 파일들. spring 설정파일, jsp, HTML, js파일등
- src/main/webapp/resources: HTML, CSS, JS 파일들
- src/main/webapp/WEB-INF/spring: spring 설정파일들
- src/main/webapp/WEB-INF/views: jsp파일들
- pom.xml : maven 설정 파일
- web.xml : 웹설정파일. DispatcherServlet 등록

web.xml
--------

- DispatcherServlet을 등록 하고 mapping(root에 등록)



- - -

동작 과정
===========

```
- 클라이언트 -> 요청->DispatcherServlet -> HandlerMapping(알맞은 controller 결정)
                           |->HandlerAdapter(Controller에서 가장 적합한 메서드 결정)
                           |<->Controller<->Service<->DAO<->DB
                           |->ViewResolver(가장 적합한 view 결정)
                           |->view -> 응답 ->client
```
- 실제로 구현하는 부분은 **Controller, Model, View**

DispatcherServlet
-------------------

- Front controller 라고도 함
- 사용자의 요청을 받는곳. HandlerAdapter와 HandlerMapping을 이용하여 Controller를 검색, 실행 한다
- web.xml에 등록
- **servlet-context.xml** 을 이용하여 초기 설정(지정하지 않으면 Spring Framework이 알아서 만듬)

```xml
<servlet>
  <servlet-name>appServlet</servlet-name>
  <servelt-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>appServlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>

<!-- HTML,CSS,JS 파일등이 담김 resource파일 사용   -->
<resources mapping="/resources/**" location="/resources/"/>
```

Controller
------------

- servlet-context.xml 에 **annotation-driven** 태그를 입력하여 사용
- 실제 사용할 클래스위에 **@Controller** 적는다

- @RequestMapping("요청한 값"): request에 맞게 HandlerAdapter가 골라냄
- model: DispatcherServlet에 의해 전달됨. View에서 가공되어 클라이언트에 전달

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;

@Controller
@ReqeustMapping("/member")//아래의 내용은 /member/ 이후로 들어온 내용을 다룬다
public class MyController{
  @RequestMapping(value="/hi" , method=RequestMethod.GET)//memeber/hi
  public String hi(Model model,HttpServletRequest request){
    String id = request,getParameter("id");
    model.setAttribute("id","Your ID is "+ id);
  }
}
```

##### @ModelAttribute("name")

- view에 변수를 넘길때 "name.value" 형식으로 넘길 수 있다

```java
  public String hi(@ModelAttribute("name") Info info,HttpServletRequest request){
    //
  }
```

- 이때 객체는 getter, setter가 정의 되어 있어야 한다
  1. HTTP로 전달되는 값이 setter를 통해서 @ModelAttribute 가 적용된 객체에 저장된다
  2. setter를 사용하거나 getter를 사용해서 처리한다
  3. view에 객체가 전달된다. 특정 이름(name).변수이름 으로 사용

- 또한 다음과 같이 @ModelAttribute 가 적용된 메서드는 다른 메서드가 호출되기 전에 호출되고 return 값을 Model 객체에 저장한다

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;

@Controller
@ReqeustMapping("/member")//아래의 내용은 /member/ 이후로 들어온 내용을 다룬다
public class MyController{
  @RequestMapping(value="/hi" , method=RequestMethod.GET)//memeber/hi
  @ModelAttribute("user")
  public String hi(Model model,HttpServletRequest request){
    String id = request,getParameter("id");
    model.setAttribute("id","Your ID is "+ id);//user.id로 접근
    return "userView"
  }
}
```

#### ModelAndView vs Model

- Model은 데이터만을 전달
- ModelAndView는 데이터와 view를 같이 전달

```java
public ModelAndView show(){
  ModelAndView mav = new ModelAndView();
  mav.addObject("id","hi");
  mav.addObject("passwd","1234");

  mav.setViewName("userView");
  return mav;
}
```

ViewResolver, View
---------------------

- Model : Controller에서 view객체에 넘기는 데이터를 포함하는 객체의 interface(실제 구현체 = ModelMap)
  - addAttribute("name","value")

```xml
<!-- servlet-context.xml 의 모습 -->
<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <beans:property name="prefix" value="/WEB-INF/views/"/>
  <beans:property name="sufix" value=".jsp"/>
</beans:bean>
```
- controller를 다음과 같이 만든다

```java
@Controller
public class MyController{
  @RequestMapping("/hi")
  public String hi(Model model){
    return "hello"//prefix+RETURN VALUE+suffix
  }
}
```

Service & DAO
-----------------

- **@Service** : 서비스 클래스 위에 적는 annotation. 적어놓으면 container에 담긴다. **@Autowired** 로 사용
- **@Componenet** : DAO 클래스 위에 적는 annotation. **@Autowired** 로 사용
