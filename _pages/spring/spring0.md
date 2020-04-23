---
layout: post
title:  "Spring Framework"
name: spring0.md
category: spring
---

Spring framework
===============

- DI, AOP, MVC, JDBC 제공
- 모듈(필요한 모듈은 XML파일로 작성해서 사용)
  1. spring-core : DI,IoC 제공
  2. spring-aop : AOP 제공
  3. spring-jdbc : DB를 쉽게 다룰 수 있는 기능 제공
  4. spring-tx : transaction 기능 제공
  5. spring-webmvc : MVC 기능 제공
- 컨테이너 : 스프링에서 여러 객체를 생성하고 이용.
- Bean : 컨테이너에 의해 생성된 객체
  - Java Bean : 데이터를 표현하는 것을 목적으로 하는 클래스. getter, setter 등을 포함
- Bean의 scope
  - singleton : 컨테이너 하나에 만든 bean은 종류마다 하나씩 생성. 기본 값
  - prototype : 하나의 Bean 정의에 다수의 객체 생성
  - request : 하나의 bean 정의로 하나의 HTTP request의 lifecycle안에 객체를 1개 생성.
  - session : 하나의 Bean 정의로 하나의 HTTP session의 lifecycle안에 객체를 1개 생성

- - -

POJO(plain old java object)
=====

- 특정 기술(환경, 규약 등)에 종속되지 않고 동작하는 순수한 자바 객체
  - 특정 기술에 종속되면 가독성, 유지보수, 확장성 등이 떨어짐.
- 객체지향에 충실해야함
- 로우레벨 코드(transaction, thread 등)와 비즈니스 코드가 분리되어서 가독성 증가
- **extends, implements** 등을 사용하는 것이 아닌 **annotation** 을 이용하여 의존성 제거
