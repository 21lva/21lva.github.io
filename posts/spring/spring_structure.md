---
layout: post
title:  "Spring 기본 내용"
name: spring_structure.md
category: spring
---

- JAVA EE, Spring, jakarta EE의 역사와 변천, Spring 기본 지식에 대해서 정리한 글

- - -

JAVA EE의 역사
============

- 원래의 서버 프로그램은 C, CPP와 다양한 회사의 미들웨어 제품들응 사용해서 개발하였지만 이 방식은 미들웨어 제품에 종속될 수 밖에 없게 됨
- 자바를 이용하면 플랫폼에 종석되지 않기 때문에 미들웨어에 필요한 공통 API를 제공하면서 문제를 해결할 수 있음
- J2EE(JAVAEE): 서버 개발에 필요한 기능을 모아서 만든 표준. 각 기업들이 해동 표준을 준수하는 미들웨어 제품을 판매하고 개발자들은 제품과 상관 없이 서버 개발을 할 수 있게 됨.
- J2EE는 Servlet, JSP등을 포함하게 되었고 PHP, ASP, CGI등을 몰아내고 인기를 얻게 됨.


- **EJB(Enterprise JAVA Beans)**: J2EE의 핵심 기술. J2EE가 대체하는 미들웨어 제품에서 기능되던 분산처리, 트랜잭션, 보안 등을 지원하는 컴포넌트 모델을 제공하는 기술
- EJB는 널리 사용되었지만 API의 모양과 플랫폼 독립성이라는 자바의 특성만을 강조한 설계에서 발생되는 몇 가지 문제가 존재
  1. EJB는 ORM 기술을 제공하였지만, 2.x 버전까지 *order by* 표준을 제공하지 않음.
  2. **sensible defaults**, **convention over configuration**같은 생각이 있기 이전이기 때문에 J2EE 서버에 배포를 하기 위해서는 많은 양의 XML 설정을 작성해야 했다.

- **Spring framework**는 EJB의 문제점을 해결하기 위해서 개발되었다. 고가의 full stack J2EE 서버가 아닌 Tomcat과 같은 일반 servlet container에서도 구동된다는 장점을 포함하였다
  - Tomcat은 J2EE의 표준의 *일부*인 Servlet의 *reference implementation*으로 출발. i.e. 실제 서비스에 사용하기 보다는 Servlet, JSP 기술을 보여주기 위한 용도로 개발되었고 점진적인 성능 개선을 통해서 실무에서 사용되는 수준으로 변모하였다.
  - Spring은 비싼 J2EE를 구매하지 않아도 복잡한 EJB보다 간단한 방식으로 EJB에서 제공하던 선언적 트랜잭션, 보안 처리, 분산 환경 지원 등의 주요 기능을 모두 사용할 수 있게 해주었다. 또한, XML 파일을 작성하는 곤욕에서 벗어날 수 있게 되었다
- 국내에서는 Spring이 매우 빠르게 퍼져나갔지만 해외에서는 J2EE가 명맥을 유지할 수 있었다. 국내는 DB 중심적인 개발 관행이 있고 JAVA로 주로 웹 UI를 만드는 데에 주력하였지만 해외에서는 J2EE 기반으로 핵심 인프라를 구축한 사례가 많았기 때문에 대규모 시스템을 Spring으로 이전하기 쉽지 않았기 때문이다. 또한, Spring의 성공 이후로 J2EE도 Spring의 성공 사례를 바탕으로 대폭적인 개선을 하였다. ORM관련 기능에서 당시 유행하던 **Hibernate** 개념을 받아들여서 **JPA**라는 표준 기술을 만들었다
  - JPA는 단순하게 Hibernate의 개념을 차용한 것이 아니라 Hibernate의 핵심 개발자가 직접 스펙 개발에 참여하면서 JPA라는 공통 표준을 만들고 Hibernate가 대표적인 구현체로서 존재하는 관계가 형성되었다.
- Spring에서도 J2EE와의 간극을 좁히기 위해서 몇 가지 노력을 수행하였다.
  - *@Autowired, @Inject*가 대표적인 사례이다. Spring에서 의존성 주입을 위해서 Autowired을 이용하였고, 이를 본 J2EE에서 Inject를 제공하기 시작했다. 그런 후에 다시 Spring에서 J2EE 표준을 지원하기 위해서 Inject를 포함하게 되었다.

- 시대가 변하면서 Servlet container 기반의 개발이 인기를 얻고 **Docker**를 이용한 MSA가 대세가 됨에 따라서 JavaEE는 점점 영향력이 줄어들었다. 이제는 **Spring Boot**와 같이 Servlet container도 포함하는 방식으로 이루어지는 중.

- - -

Spring
======

- spring framework: java enterprise 개발을 도와주는 application framework
  - 스프링은 특정 계층, 기술, 분야에 국한되지 않고 어플리케이션 전 영역에서 사용가능한 프레임워크. 어플리케이션의 전 영역을 관통하는 일관적인 프로그래밍 모델(MVC, JDBC 등)과 기술(IoC, DI, AOP 등)을 제공.
  - 스프링은 불필요하게 무겁지 않다. EJB에 적용된 과도한 엔지니어링 기술(무거운 WAS, 불필요한 패키징 등)을 지양한다. 스프링은 단순한 WAS인 서블릿 컨테이너(Tomcat, jetty 등)에서 동작.
  - 스프링은 근본적인 부분에서 자바 엔터프라이즈 개발을 쉽게 해준다. 어플리케이션 개발자들이 스프링에서 제공하는 기술들의 동작원리는 알지 못해도 비즈니스 로직에 집중할 수 있도록 해줌.

스프링을 쓰는 이유
--------------

- 스프링을 제대로 사용하기 위해서는 스프링의 목적을 알아야 한다
- 많은 사람들은 JavaEE가 실패한 이유를 **너무 복잡해서**라고 대답한다.
  - 엔터프라이즈 시스템은 서버에서 동작하면서 기업의 업무 처리를 해주는 시스템이다. 그렇기 때문에 효율적이고 보안이 뛰어나야 하는 등의 많은 요구사항을 만족해야 한다. 이러한 요구사항은 무거운 WAS나 툴을 사용한다고 충족되는 것이 아니다. 복잡한 기술들이 들어간다. 또한, 요구사항 종류 뿐만 아니라 요구사항 자체의 복잡함도 증가하였다.
  - 기술적, 비즈니스적 복잡함은 제거 대상이 아니고 오히려 달성해야할 목표이다. 복잡함을 제거하기 보다는 복잡함을 편리하게 만족시킬 수 있는 기술이 필요하다는 말이다.
- EJB는 어플리케이션 핵심 로직에서 기술적인 복잡함을 분리하는데에는 성공하였다. 그러나 특정 인터페이스를 구현하거나 상속 받기도 해야하고 서버에 종석적인 서비스를 통해서만 사용가능하게 만드는 등의 한계가 존재하였다. 복잡함을 해소하기 위해 새로운 복잡함을 만든 것이다.
- 스프링은 EJB와 다르게 접근하였다. EJB가 했던 기술적인 복잡함을 어플리케이션의 로직에서 제거하면서 비침투적인 방법으로 스프링 기술과 코드를 복잡하게 만드는 것을 최대한 방지하려 하였다.
- 스프링이 기술적인 복잡함을 여러 방식으로 해결하였다
  - 기술에 접근하는 일관성 있는 방법 제공: 서비스 추상화, 트랜잭션 추상화와 같은 기술을 사용하여 인터페이스를 분리하고 세부 기술에 독립적인 인터페이스를 제공한다
  - 기술관련 코드가 다른 코드에 스며들지 않게 한다: AOP를 통하여 어플리케이션 로직을 담당하는 코드에 남아있는 기술적 코드를 최대한 분리한다.
- 기술적인 복잡함과 비즈니스적 복잡함을 제거하고 나면 어플리케이션 개발자는 비즈니스적 복잡함을 최대한 어플리케이션에서 개발하게 된다. 기술적인 복잡함을 제거하기 위해 스프링은 DI를 통해서 자바객체지향의 장점을 극대화할 수 있게 해준다. 그 외의 비즈니스 로직의 복잡함은 DI보다는 객체지향적 설계가 중요하다. 스프링은 객체지향 분석과 설계가 도메인 모델에 쉽게 적용될 수 있도록 기술적인 코드와 비즈니스 로직을 분리한다.

POJO(plain old java object)
----------------

- 스프링의 장점으로는 엔터프라이즈 서비스 기능(보안, 트랜잭션 등의 기능적 요구사항)을 **POJO**(어플리케이션 로직을 담은 코드)에 제공하는 것이다.
- 스프링으로 개발한 어플리케이션은 POJO를 이용하여 정의한 어플리케이션 코드 + POJO들의 관계를 정의해놓은 설계정보로 구분된다. DI는 POJO 오브젝틀르 설계정보대로 동적으로 주입해주는 것이다. IoC/DI, AOP, PSA등은 POJO로 어플리케이션을 개발할 수 있게 해주는 *enabling technology*이다.

- POJO: 특정 기술(환경, 규약 등)에 종속되지 않고 동작하는 순수한 자바 객체로 객체지향 설계원칙을 따라야한다. 특정 기술에 종속되면 가독성, 유지보수, 확장성 등이 떨어짐.

- - -

IoC(Inversion of Control)
=====================

- Java Bean : 데이터를 표현하는 것을 목적으로 하는 클래스. default 생성자와 프로퍼티(getter, setter가 설정됨)을 포함
- spring에서의 Bean : 스프링의 컨테이너에 의해 생성된 객체
- Bean의 scope
  - singleton : 컨테이너 하나에 만든 bean은 종류마다 하나씩 생성. 기본 값
  - prototype : 하나의 Bean 정의에 다수의 객체 생성
  - request : 하나의 bean 정의로 하나의 HTTP request의 lifecycle안에 객체를 1개 생성.
  - session : 하나의 Bean 정의로 하나의 HTTP session의 lifecycle안에 객체를 1개 생성
- 컨테이너: 객체의 lifecycle을 관리하고 생성된 객체에게 추가적인 기능을 제공. Bean 객체의 lifecyle = 스프링 container와 같음
- 스프링 컨테이너 : DI를 이용하여 application을 구성하는 요소들을 관리. **BeanFactory, ApplicationContext** 가 존재

- IoC(inversion of Control,제어역전): 오브젝트가 자신이 필요로하는 오브젝트를 생성하는 구조가 아니라 역으로 필요한 다른 오브젝트의 lifecycle을 관리하는 관리자가 제공해주는 구조. ex) framework는 라이브러리와 달리 어플리케이션 코드가 프레임워크에 의해 사용되는 구조이다.
- 컨테이너를 쓰지 않는 일반적인 java 프로그램의 경우 프로그래머가 직접 객체를 생성하는 코드를 넣고 수정해야 하지만, 컨테이너를 이용하면 이 과정을 대신(**위임**) 해준다.
- 컨테이너를 사용하면 다음과 같은 장점이 있다
  1. 사용자는 구체적인 factory class와 사용법을 알 필요 없다
  2. 컨테이너가 자동생성, 후처리 등과 같은 기능을 제공해준다
  3. 컨테이너가 bean을 검색하는 기능을 제공한다

BeanFactory
-------------

- DI처리, Bean(객체)의 등록,생성 조회, 반환을 담당
- BeanFactory계열의 interface만 구현한 클래스
- lazy loading: BeanFactory는 Bean의 내용(정의)는 즉각 로드하지만, 필요할 때까지 instance를 생성하지는 않는다
- getBean() 메서드를 통해서 instance화 한다

ApplicationContext
--------------------

- BeanFactory에 여러가지 기능을 추가한 컨테이너. 보통 BeanFactory보다 많이 사용한다
  - 국제화가 지원되는 text message 관리
  - image 로드 같은 포괄적인 방법 제공
  - listener로 등록된 bean에게 이벤트 발생을 알려줌
- context 초기화 시점에 모든 singleton bean을 로드하고 생성해서 Bean을 지연 없이 사용 가능

- 실제 사용하는 컨테이너
  - ClassPathXmlApplicationContext : classpath에 위치한 xml 파일에서 context 내용을 읽음
  - FileSystemXmlApplicationContext: 경로를 정해서 context 내용 읽고 스프링 컨테이너 생성
  - XmlWebApplicationContext: 웹 어플리케이션에 포함된 xml 파일에서 context 내용 읽음
  - GenericXmlApplicationContext: Xml 파일을 설정정보로 사용하는 스프링 컨테이너 클래스
  - AnnotationConfigApplicationContext: **Groovy** 로 작성된 설정정보를 이용하는 스프링 컨테이너 클래스
  - AnnotationConfigWebApplicationContext: 웹 어플리케이션을 개발할 때 사용하는 스프링 컨테이너. 자바코드를 설정정보로 사용

- - -

DI(Dependency Injection)
====

Dependency
----------

- A라는 클래스가 B라는 클래스를 내부에서 사용하면 A는 B에 **의존적** 이다
- Aggregation(집합관계) : 부분이 전체와 다른 lifecycle을 가짐 ex) 집과 컴퓨터
- Composition(구성관계) : 부분이 전체와 같은 lifecycle을 가짐 ex) 노트북, CPU

DI
---

- 의존 관계를 설정해주는 일(생성자로 설정하고 setter를 이용하여 변경)
- 스프링에서 DI : 의존관계를 A가 B의 interface에 의존하게 한 후에 실제 클래스 할당은 컨테이너에 의해 수행됨
- 의존하고 있는 클래스가 변경되어도 클래스가 아닌 인터페이스에 의존하고 있기 때문에 쉽게 수정사항을 반영할 수 있다. 결함성이 감소
- DI의 기능들
  1. 기능 변경: 필요에 따라 주입되는 대상(인터페이스 구현 대상들)을 변경. 이 변경은 런타임 중에 동적으로 이루어질 수도 있다.
  2. 기능 추가: 데코레이터 패턴, AOP와 같은 기능을 쉽게 적용할 수 있다.
  3. 인터페이스 변경: 클라이언트가 사용하는 인터페이스와 오브젝트가 상속한 인터페이스 사이에 차이가 있으면, 어댑터 오브젝트를 만들어서 DI를 해주면 된다. 이를 조금 더 일반화하면 PSA
  4. 프록시: 필요한 시점에 실제 사용할 오브젝틀르 초기화하고 지연 로딩을 적용하기 위해서는 프록시가 필요.
  5. 템플릿, 콜백 패턴
  6. 오브젝트 스코프: DI를 통해서 오브젝트의 lifecyle을 관리할 수 있다.
  7. 테스트: 테스트 하는 대상에 필요한 test double을 제공하거나 필요에 따라 다른 대상을 주입할 수도 있다.

DI 방법
--------

- getter와 setter를 사용하는 방법 외에도 3가지의 방법을 spring에서 제공한다

#### 1) XML 사용

- resource 디렉토리 밑에 xml파일을 만든다
- property를 이용해서 멤버변수의 setter에 default 값을 제공한다

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--
FILENAME : applicationContext.xml
-->
 <bean id="Apple" class="pkg1.Apple"/>
 <bean id="Melon" class="pkg1.Melon"/>
 <bean id="ShopList" class="pkg1.ShopList">
   <property name="fruit" ref="Melon"></property>
 </bean>
</beans>
```

- java 파일에 의존성 부분을 다음과 같이 작성한다

```java
package pkg1;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class Shopper{
 public static void main(String[] args) {
  ApplicationContext context = new FileSystemXmlApplicationContext("/src/main/java/pkg1/applicationContext.xml");
  ShopList shopList= (ShopList)context.getBean("ShopList");

  Fruit fruit = (Fruit)context.getBean("Apple");
  shopList.setFruit(fruit);

  System.out.println(car.getTireBrand());
 }
}
```

#### 2) Autowired

- xml을 사용하지만 property, contructor-org 등을 사용하지 않고 자동으로 의존 대상을 찾아서 주입
- xml에 상단 부분이 추가된 설정이 있다(**중요**)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:context="http://www.springframework.org/schema/context"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context-3.1.xsd">


<!--
FILENAME : applicationContext.xml
-->
  <context:annotation-config/>
 <bean id="Apple" class="pkg1.Apple"/>
 <bean id="Melon" class="pkg1.Melon"/>
 <bean id="ShopList" class="pkg1.ShopList"/>
</beans>
```

- Shopper가 아닌 **ShopList클래스** 를 수정한다
  - Autowired된 Fruit 부분이 자동으로 설정된다

```java
package pkg1;

import org.springframework.beans.factory.annotation.Autowired;

public class ShopList{
  @Autowired
  Fruit fruit;
}
```

- Autowired알고리즘
  1. 해당 type를 구현(implement)한 bean이 없으면, **No Matchting bean Error**. 있으면 2로
  2. Bean이 둘 이상이고 name(멤버변수의 이름)과 일치하는 id가 하나가 아니면, **No Unique bean Error**.
  3. 다 통과하면 해당 bean을 변수에 할당

- Bean이 여러개이면 **@Qualifier(값)** 을 통해서 지정할 수 있다
  ```xml
  <bean class="asdcsa.ascac">
    <qualifier value="cosmetics"/>
  </bean>
  ```

- [Field injection, Constructor injection](https://zorba91.tistory.com/238)
- [Field injection, setter injection, constructor injection](https://leejisoo860911.tistory.com/2)

#### 3) @Resource 이용

- 기본적으로 **@Autowired** 와 비슷
- 차이점은 **@Autowired** (Spring만의 사용법) 와 달리 JAVA 표준이다.
- 또한, 알고리즘 순서도 type -> name 이 아닌 name->type이다

#### 4) @Inject

- @Autowired 와 비슷하게 사용
- 차이점 : 해당 객체가 만들어 있지 않아도 사용할 수 있게 해주는 **@Autowired(required=false)** 를 지원하지 않음
- **@Named(value="bean객체의 id")** 와 같이 사용해서 특정 bean 사용 가능(@Qualifier와 비슷)


#### 5) [다양한 의존 주입 방법](https://engkimbs.tistory.com/683)


- - -

설정파일 분리
=============

- 설정파일이 커졌을때를 나누는 것
- 기능별로 xml 설정 파일을 분리해서 사용

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--
FILENAME : applicationContext.xml
-->

<import resource="/src/main/resource/app1.xml"/>
<import resource="/src/main/resource/app2.xml"/>

  <context:annotation-config/>
 <bean id="Apple" class="pkg1.Apple"/>
 <bean id="Melon" class="pkg1.Melon"/>
 <bean id="ShopList" class="pkg1.ShopList"/>
</beans>
```

- 또는 아래와 같이 여러 xml을 java에서 배열로 설정해서 사용할 수 있다

```java
package pkg1;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class Shopper{
 public static void main(String[] args) {
   String ctxFiles = ["/src/main/java/resources/app1.xml","/src/main/java/resources/app2.xml"];
  ApplicationContext context = new FileSystemXmlApplicationContext(ctxFiles);
  ShopList shopList= (ShopList)context.getBean("ShopList");

  Fruit fruit = (Fruit)context.getBean("Apple");
  shopList.setFruit(fruit);

  System.out.println(car.getTireBrand());
 }
}
```

- - -

Spring MVC와 서블릿
==============

CGI, Servlet, JSP
----------------

- CGI(Common Gateway Interface): 사용자가 동적으로 데이터를 요청할 때 서버에서 다른 프로그램을 동작시키고 그 결과를 전달해주는 역할을 위한 표준적인 방법. 서버와 외부 응용 프로그램을 연결해주는 역할. 단순 interface이기 때문에 다양한 언어로 실제 구현체를 작성하였다.
- JSP: HTML에 Java 코드를 넣어서 쓸 수 있게 해주는 기술. 일종의 서버사이드 스크립트이다. Java Servlet으로 변환되어 실행되기 때문에 Servlet과 유사한 역할을 수행한다.

- Servlet: CGI에서 이루어지는 동작을 Process 단위가 아닌 Thread 단위로 이루어질 수 있도록 하는 역할을 수행. Servlet을 갖고 있는 Container가 데이터를 주고 받는 역할을 한다. 각각의 Servlet은 Servlet interface를 구현하고 이 구현체들을 Servlet Container가 갖고 있다.
  - 사용자가 요청을 전달하면 요청을 컨테이너가 *HttpServletRequest, HttpServletResponse* 객체를 생성하고 적당한 Servlet에 요청을 전달한다. Servlet에서는 적당한 요청 방식(POST,GET 등)에 맞게 동작을 수행하고 결과를 *HttpServletResponse*에 결과를 저장하고 컨테이너에 전달한다. 컨테이너는 해당 객체를 바탕으로 Response를 사용자에게 전달한다.
  - Servlet Container는 사용자의 요청이 들어오면 Servlet이 메모리에 있는지 확인하고 없으면 생성한다. 클라이언트 요청에 따라서 적당한 Servlet의 *service* 메서드를 호출하고 이 때 *HttpServletRequest, HttpServletResponse* 객체를 같이 전달한다. Servlet들은 보통 Singleton으로 관리.
  - Container는 Servlet의 생명주기를 관리하면서 클라이언트의 요청을 송수신하기 위해서 웹서버와 socket 통신을 한다. 요청마다 자바 쓰레드를 생성하면서(혹은 ThreadPool에서 가져온다) 멀티쓰레드를 지원 및 관리한다. 서블릿 자체는 스레드끼리 공유 Tomcat이 대표적인 Servlet Container의 한 예시.
  - WAS에서도 자체적으로 웹서버를 내장하고 있다. WAS앞의 웹서버는 로드밸런싱, 방화벽 등의 기능을 담당한다. 즉, 이 기능이 필요없으면 없어도 된다는 말. 웹서버의 예시로는 apache web server, nginx가 존재
  - 클라이언트의 요청 -> 웹서버 -> WAS내의 웹서버 -> WAS내 서블릿 컨테이너 -> 서블릿 -> WAS내 서블릿 컨테이너 -> WAS내 웹서버 -> 웹서버 -> 클라이언트

- spring MVC에 있는 DispatcherServlet는 서블릿 중 하나이다. DispatcherServlet은 서블릿 컨테이너로부터 전달 받은 요청을 controller에게 전달한다. 그 과정은 다음과 같다.

```
- 클라이언트 -> 요청 -> ServletContainer -> DispatcherServlet -> HandlerMapping(알맞은 controller 결정)
                                                           |-> HandlerAdapter(Controller에서 가장 적합한 메서드 결정)
                                                           |<-> Controller <-> Service <-> DAO <-> DB
                                                           |-> ViewResolver(가장 적합한 view 결정)
                                                           |-> View -> 응답 ->client
```

- 실제로 개발자들이 구현하는 부분은 **Controller, Model, View**

Spring MVC에서의 Servlet Container, Spring Container
--------------------------------------

1. Web Application이 동작하면 Servlet Container에 의해서 web.xml 파일이 로딩된다. Servlet Container는 프로세스 당 하나만 동작한다
2. Servlet Container는 web.xml에서 *ContextLoaderListener* 정보를 읽어서 객체화한다.
  - *ContextLoaderListener*: *ServletContextListener*를 구현. root-content.xml or ApplicationContext.xml을 읽어서 *ApplicationContext*를 생성.
  - *ApplicationContext*: Bean이 등록되어 관리되는 Servlet. Servlet Context에서 Spring Bean들에 대한 정보를 얻기 위해 접근하는 장소. Spring Container가 구동되면서 여러 Bean들과 객체들이 생성된다.
3. 클라이언트로부터 요청이 들어오면 다음과 같은 과정이 이루어진다
  3-1. *DispatcherServlet*이라는 Servlet이 Servlet Container에 의해서 한 번 생성된다. 이 Servlet은 요청마다 생성되는 *Thread*들이 공유한다. 웹서버로 들어온 모든 요청은 Servlet Container로 전달되도록 설정되어 있고 다시 그 요청들은 전부 *DispatcherServlet*으로 전달되도록 설정되어 있다.
  3-2. Servlet Container는 요청이 들어오면 thread을 생성하고 thread에서 DispatcherServlet을 사용한다.
  3-3. 요청이 *DispatcherServlet*에 전달된 후에는 위에 작성된 과정을 거쳐 로직이 수행된다.

- Spring container(e.g. ApplicationContext)도 하나의 Servlet이며 Servlet Container에 의해 관리된다.
- Spring MVC 이전에는 URL마다 Servlet이 생성하도록 web.xml를 작성해야했지만 DispatcherServlet 이후로는 Fron Controller 패턴을 이용해서 그럴 필요가 없게 되었다.

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

- - -

Reference
=======

- https://okky.kr/articles/415474
- https://www.samsungsds.com/kr/insights/java_jakarta.html
- [스프링의 역사 참조 자료](https://velog.io/@hanblueblue/%EA%B2%8C%EC%8B%9C%ED%8C%90-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-SpringMVC-Gradle-MySql-JPA-1.-Spring%EC%99%80-SpringMVC)
- [참고가 많이 된 자료](https://expert0226.tistory.com/189)
- [Servlet Container vs Spring Container](https://jypthemiracle.medium.com/servletcontainer%EC%99%80-springcontainer%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4-%EB%8B%A4%EB%A5%B8%EA%B0%80-626d27a80fe5)
- [ServletContainer와 SpringContainer는 무엇이 다른가? | by Sigrid Jin | Medium](https://sigridjin.medium.com/servletcontainer%EC%99%80-springcontainer%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4-%EB%8B%A4%EB%A5%B8%EA%B0%80-626d27a80fe5)
