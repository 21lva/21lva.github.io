---
layout: post
title:  "Annotation을 이용한 설정"
name: spring_annotation.md
category: spring
---

Annotation을 이용한 설정
======================

- XML 대신 java파일로 설정파일을 만든다

작성
----

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeanConfig{

  //<bean id="apple" class="pkg1.Apple"/>
  @Bean
  public Apple apple(){
    //return type : class
    //method name : id
    return new Apple();
  }

  //init-method 사용의 경우
  @Bean(initMethod="init")
  public Apple apple(){
    return new Apple();
  }

  //특정 생성자를 사용하는 경우
  @Bean
  public Apple apple(){
    return new Apple(color());
  }

  //property 있는 bean
  @Bean
  public Apple apple(){
    Apple ret = new Apple();
    ret.setColor("Blue");
    ArrayList<String> makers = new ArrayList<String>();
    makes.add("Harry");
    makes.add("Bob");
    ret.setMakers(makers);
    return ret;
  }
}
```

사용
----

- xml 설정파일을 사용하는 부분에 다음과 같이 적는다

```java
//GenericXmlApplicationContext ctx = new GenericXmlApplicationContext("classpath:applicationCTX.xml")

AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(BeanConfig.class);
```

설정 파일 분리
-------------

- 다음의 파일을 분리해보자

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeanConfig{
  @Bean
  public Color color(){
    return new Color();
  }

  @Bean
  public Apple apple(){
    Apple ret = new Apple();
    ret.setMakers(color());
    return ret;
  }
}
```

- 1번 파일

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class ColorConfig{
  @Bean
  public Color color(){
    return new Color();
  }
}
```

- 2번 파일
  - 하나의 컨테이너에 모든 객체를 자동으로 주입하기 때문에 **@Autowired** 를 통해서 import 하지 않은 Bean을 사용할 수 있다

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppleConfig{
  @Autowired
  Color color;

  @Bean
  public Apple apple(){
    Apple ret = new Apple();
    ret.setMakers(color);
    return ret;
  }
}
```

- 사용

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppleConfig.class,ColorConfig.class);
```

@Import
--------

```java
import org.springframework.context.annotation.Configuration;

@Configuration
@Import({AppleConfig.class,ColorConfig.class})
public class MemberConfig{
  //생략
}
```

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(MemberConfig.class);
```
