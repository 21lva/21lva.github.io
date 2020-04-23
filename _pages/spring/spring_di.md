---
layout: post
title:  "DI"
cover:  "/assets/instacode.png"
name: spring_di.md
---

DI(Dependency Injection)
====

- [참고가 많이 된 자료](https://expert0226.tistory.com/189)

Dependency
----------

- A라는 클래스가 B라는 클래스를 내부에서 사용하면 A는 B에 **의존적** 이다
- Aggregation(집합관계) : 부분이 전체와 다른 lifecycle을 가짐 ex) 집과 컴퓨터
- Composition(구성관계) : 부분이 전체와 같은 lifecycle을 가짐 ex) 노트북, CPU

DI
---

- 의존 관계를 설정해주는 일(생성자로 설정하고 setter를 이용하여 변경)
- 스프링에서 DI : 의존관계를 A가 B의 interface에 의존하게 한 후에 실제 클래스 할당은 컨테이너에 의해 수행됨
- 의존하고 있는 클래스가 변경되어도 클래스가 아닌 인터페이스에 의존하고 있기 때문에 쉽게 수정사항을 반영할 수 있다

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

참고자료
===========

- <https://jjunii486.tistory.com/84>
