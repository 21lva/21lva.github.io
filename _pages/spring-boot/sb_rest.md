---
layout: post
title:  "REST"
name: sb_rest.md
category: spring-boot
---

- REST(Reprenstational State Transfer) : REST

REST
=====

- Reprenstational State Transfer : 통신 네트워크 아키텍처. 통신하는 리소스끼리 복잡하지 않은 방식으로 상호작용하기 위함
- HTTP를 이용하여 URI + GET, POST, PUT , DELETE
  - URI : 인터넷 상의 특정 자원을 나타내는 주소값
- RESTful : REST 원칙을 따르는 것

인터페이스 일관성
----------------

- 자원 식별(URI) : URI을 이용한 자원 식별. **정보/유일자** 형식
- 메시지를 통한 리소스 조작 : 메타 데이터를 가지고 자원을 수정하거나 삭제하는 정보를 제공. ex) content-type
- 자기 서술적 메시지 : 처리 방법에 대한 서술. GET, PUT, POST 등
- HATEOAS : 클라이언트에 응답할 때 단순히 결과만이 아닌 URI를 함께 제공해야한다는 원칙

REST API
----------

- 구성요소
  1. 자원(URI)
  2. 행위(HTTP 메서드. CRUD)
    - GET : 요청. read
    - POST : 제공. create
    - PUT : 수정. update
    - DELTE : 삭제. delete
  3. 표현(HTTP Message Body)

```
GET http://localhost:8080/api/books/1
content-type: application/json
```

- - -

@JsonIgnore
------------

- 해당 필드는 **GET** 메서드의 반환값에 포함되지 않는다


```java
@Column
@JsonIgnore
private string password;
```

@Projection
------------

- 특정 상황에서만 사용
- proprety
  - name : 해당 객체가 사용되기 위한 key
  - types: 어떤 domain에서 사용할지를 정의

```java
@Projection(name="getOnlyName", types={User.class})
public interface UserOnlyConatinName{
  String getName();
}
```

- 사용하려면 다음과 같이 한다

```java
@RepositoryRestResource(excerptProjection=  UserOnlyContainName.calss)
public interface UserRepository extends JpaRepository<User,long>{
//생략  
}
```

- - -

메서드 권한 설정
==============

- 특정 메서드를 권한에 맞게 사용할 수 있게 해준다
- 아래의 메서드들을 사용하려면 configure class에 **@EnableGlobalMethodSecurity(secruedEnable=true,prePostEnabled=true)** 를 이용한다

@Secured
---------

- 권한별로 접근 권한 설정
- OR 설정만 가능. 표현식 불가능

```java
@Secured({"role1","role2"})
@GetMapping("/api/secureRole")
public String secureRole() throws Execption{
  return "This is a secureRole function"
}
```

@PreAuthorize
------------

- 권한별로 접근 권한 설정
- 표현식을 사용할 수 있다

```java
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/api/preRole")
public String preRole() throws Execption{
  return "This is a preRole function"
}
```

- 사용해보자

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled=true)
@EnableSecurity
static class SecurityConfiguration extends WebSecurityConfigurerAdapter{
  @Bean
  InMemoryUserDetailsManager userDetailManager(){
    User.UserBuilder commonUser =User.withUsername("commonUser").password("{noop}common").roles("USER");
    User.UserBuilder havi =User.withUsername("havi").password("{noop}test").roles("USER","ADMIN");

    //생략
  }
}
```
