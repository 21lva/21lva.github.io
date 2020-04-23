---
layout: post
title:  "Life Cycle"
name: spring_lifecycle.md
---

Life Cycle
==========

초기화 메서드
-----

- Bean 객체를 생성하고 DI를 한 후에 실행되는 메서드
  - Bean 객체의 생성자 메서드 -> DI -> 초기화 메서드
- 초기화 메서드 실행 순서 : @PostConstruct -> InitializingBean -> init-method


#### 1) Implement InitializingBean interface

- Spring에서 제공하는 **InitializingBean** interface의 **afterPropertiesSet** 메서드를 오버라이딩 한다

```java
public class Apple implements InitializingBean{
  @Override
  public void afterPropertiesSet() throws Exception{
    System.out.println("After DI");
  }
}
```

#### 2) @PostConstruct

- 원하는 메서드 위에 **@PostConstruct** annotation을 적으면 Spring이 자동으로 초기화 메서드를 호출해줌
- 매개변수, 반환값이 존재하면 안된다

```java
public class Apple{
  @PostConstruct
  public void myPostConstruct(){
    System.out.println("After DI");
  }
}
```

#### 3) xml을 이용하여 init-method 등록

- Bean에 초기화 메서드를 등록할 때 사용가능

```xml
<bean id="Apple" class="pkg1.Apple" init-method="init"/>
```

```java
public class Apple{
  public void init(){
    System.out.println("After DI");
  }
}
```

제거 메서드
---------

- 컨테이너가 종료될 때 호출. 리소스 반환 등의 작업을 수행
- 실행 순서 : @PreDestroy -> DispoableBean -> detroy-method

#### 1) Implement DispoableBean interface

- Spring에서 제공하는 **DispoableBean** interface의 **destroy** 메서드를 오버라이딩 한다

```java
public class Apple implements DispoableBean{
  @Override
  public void destroy() throws Exception{
    System.out.println("Before eliminate this Bean");
  }
}
```

#### 2) @PreDestroy

```java
public class Apple{
  @PreDestroy
  public void myPreDestroy(){
    System.out.println("After DI");
  }
}
```

#### 3) xml을 이용하여 init-method 등록

- Bean에 제거 메서드를 등록할 때 사용가능

```xml
<bean id="Apple" class="pkg1.Apple" destroy-method="destroy"/>
```

```java
public class Apple{
  public void destroy(){
    System.out.println("After DI");
  }
}
```

- - -

참고자료
==========

- <http://wonwoo.ml/index.php/post/1820>
