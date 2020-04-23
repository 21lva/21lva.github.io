---
layout: post
title:  "IoC"
cover:  "/assets/instacode.png"
name: spring_ioc.md
---

IoC
====

- 컨테이너: 객체의 lifecycle을 관리하고 생성된 객체에게 추가적인 기능을 제공
  - Bean 객체의 lifecyle = 스프링 container와 같음
- 스프링 컨테이너 : DI를 이용하여 application을 구성하는 요소들을 관리.
  - **BeanFactory, ApplicationContext** 가 존재

- IoC(inversion of Control,제어역전): 컨테이너를 쓰지 않는 일반적인 java 프로그램의 경우 프로그래머가 직접 객체를 생성하는 코드를 넣고 수정해야 하지만, 컨테이너를 이용하면 이 과정을 대신(**위임**) 해준다.


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
