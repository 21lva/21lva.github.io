---
layout: post
title:  "@SpringBootApplication, @Component"
name: sb_annotation.md
category: spring-boot
---

SpringBootApplication
=====================

- Configuration + EnableAutoConfiguration + ComponentScan

ComponentScan annotation
---------------------------

- **ComponentScan** annoation : **Component**(Configuration도 해당) 와 **stereotype(Service, Repository, Controller)** annotation이 부여된 class를 자동으로 **Bean** 으로 등록해줌
- 이전에 xml파일에서 bean을 등록하거나, config class를 생성하여 등록하는 과정을 자동으로 해줌

#### 범위

- 기본 : 현재 class가 존재하는 패키지와 그 하위 패키지를 전부 등록

- 또는 아래와 같이 **basePackages** 혹은 **basePackageClasses** 를 사용
  - basePackages : 직접 패키지 경로를 작성.
  - basePackageClasses : 해당 클래스가 존재하는 패키지와 그 하위 패키지 등록

```java
//basePackages
@Configuration
@ComponentScan(basePackages = "com.demo.pkg1")

//basePcakgeClasses
@Configuration
@ComponentScan(basePackageClasses = userInput.class)
```

EnableAutoConfiguration
------------------------

- Spring boot의 중요한 부분.
- **META-INF** 의  **spring.factories** 에 정의 되어 있는 Bean들을 자동으로 등록해줌

- [적용 방법](https://velog.io/@max9106/Spring-Boot-EnableAutoConfiguration), [적용 2](https://engkimbs.tistory.com/753), [참고자료](http://dveamer.github.io/backend/SpringBootAutoConfiguration.html) 를 읽어보자

- - -

@Bean, @Component
====================

@Component
---------

- stereotype : classpath scanning을 통해서 bean을 관리하기 쉽게 해준 annotatino들.
  - Repository, Service, Controller, Component 등이 존재
- 클래스 상단에 적으며 **클래스 이름** 이 bean의 이름이 된다.
- CompnentScan을 통해서 자동으로 찾고 관리해주는 bean이다.

@Bean
----

- Configuration annotation 으로 선언된 class 내에 있는 메소드를 통해서 정의할때 사용.
- 메소드가 반환하는 객체가 bean이 되며, **메소드 이름** 이 bean의 이름이 된다.
- 제어가 힘든 외부 라이브러리등을 사용할 때 이용(config class 선언)

```java
@Configuration
public class myConfig{
  @Bean(name="newArray")
  public ArrayList<String> array(){
    return new ArrayList<String>();
  }
}
```

- - -

객체 주입
==========


@Autowired
---------

- bean을 자동으로 주입해주기 위해 사용
- 타입 -> 이름 -> Qualifier -> 실패

@Resource
-----------

- bean을 자동으로 주입해주기 위해 사용
- 이름 -> 타입 -> Qualifier -> 실패

@Inject
-----------

- bean을 자동으로 주입해주기 위해 사용
- 이름 -> 타입 -> Qualifier -> 실패


@Primary
--------
- Bean, Component annotaion과 함께 사용
- 객체 생성의 우선권 보장

@Qualifier
-----------

- Autowired annotation과 함께 사용
- Autowired 사용시 Bean의 이름을 지정해주어서 사용하게 함
- 클래스 정의 위에 사용하여 Bean의 이름을 지정할 수도 있음

- - -

참고 자료
==========

- [다양한 Annotation][https://jeong-pro.tistory.com/151]
- <https://jhleed.tistory.com/126>
- <https://cornswrold.tistory.com/314>
- <https://galid1.tistory.com/494>
