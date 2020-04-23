---
layout: post
title:  "lombok"
name: sb_lombok.md
category: spring-boot
---

Lombok
==========

- [공식 홈페이지](https://projectlombok.org/)
- vo,dao 등의 클래스에 getter, setter ,toString 등의 메서드를 자동으로 정의해주는 라이브러리

ToString
---------------

- 필드 이름과 값이 출력됨
- **exclude** 를 통해서 특정 변수(필드)는 제외하기 가능

```java
@ToString(exclude = "password")
public class User {
  private Long id;
  private String username;
  private String password;
  private int[] scores;
}

User user = new User();
System.out.println(user);//클래스이름(필드1=값1,필드2=값2)
```

EqualsAndHashCode
------------------

- **equals, hashCode** 메서드를 생성
  - equals : 부모클래스를 포함한 필드 값이 동일한지 확인. callSuper=false를 통해서 현재 클래스의 필드값만 비교

Getter, Setter
-----------

RequiredArgsConstructor
-----------------------

- 모든 멤버 변수를 초기화하는 생성자 만듬

Data
----

- Getter, Setter, EqualsAndHashCode, ToString, RequiredArgsConstructor 의 내용을 모두 포함하는 annotation

- - -

참고자료
=========

- <https://www.daleseo.com/lombok-popular-annotations/>
