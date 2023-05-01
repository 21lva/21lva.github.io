---
layout: post
title:  "AOP"
name: spring_aop.md
category: spring
---

Aspect Oriented Programming
============

- 트랜잭션을 적용해보자 보통은 다음과 같은 과정을 거친다
  1. 비즈니스 로직과 트랜잭션을 처리하는 기능을 분리한다.
  2. 데코레이터 패턴을 이용하여 비즈니스 로직을 담당하는 서비스를 감싸는 트랜잭션 처리 서비스를 만든다. 이로써 비즈니스 로직을 담당하는 서비스에슨 트랜잭션 코드가 없어지게 된다
  3. 데코레이터 패턴을 위해서 비즈니스 서비스마다 트랜잭션 기능을 부여하는 코드나 단순히 위임하는 코드를 만드는 것은 불편하다. JDK의 다이나믹 프록시 기능을 이용해서 런타임시에 프록시 오브젝트를 만들 수 있다. 그러나 동일한 기능의 프록시를 각각의 오브젝트에 대해 만드는 것은 해결하지 못함.
  4. 스프링 컨테이너의 bean 생성 후처리 기법을 이용하여 컨테이너의 초기화시에 자동으로 프록시를 생성하는 방법을 도입. 포인트컷을 사용하여 어떤 클래스에 트랜잭션 기능을 적용할지 선정 가능.

- 위에서 하고자하는 일은 트랜잭션 기능을 담당하는 모듈을 만드는 것이다. 많은 사람들은 트랜잭션 기능의 모듈화 작업은 OOP와 구분되는 방법으로 진행되어야 한다고 생각했다. 이 때 등장한 개념이 **Aspect(관점)**이다. aspect는 핵심기능은 아니지만 다른 핵심기능과 함께 쓰이고 여러 모듈에서 사용하는 모듈을 의미한다.
- AOP: 비즈니스 로직과 기술적인 공통 로직을 분리해서 Aspect라는 모듈로 설계하고 공통 로직을 필요한 때에 불러오는 프로그래밍 방법. 어플리케이션을 특정한 관점에서 바라보는 의미.
- spring aop 적용 예시: 로깅, 권한 체크, 트랜잭션 등


AOP 적용 기술
==============

- 스프링은 IoC/DI, Dynamic proxy, decorator pattern, auto proxy creation 드으이 기능을 이용하여 AOP를 지원. 이 중 가장 핵심은 proxy로 DI로 연결된 bean 사이에 프록시가 들어가서 부가 기능을 제공한다.
- runtime weaving: spring aop에서 프록시 생성시에 인터페이스가 존재하는 대상은 JDK Dynamic proxy를 생성하고, 없을 시에는 CBLib를 활용하여 프록시를 생성한다. spring boot에서는 기본으로 CGLIB 사용(proxyTarget=true로 기본 설정되어있다).
  - JDK Dynamic Proxy: 자바의 리플렉션 패키지의 Proxy 클래스를 이용하여 ProxyFactory에서 프록시 객체 생성. 인터페이스가 필요. 리플렉션을 이용하기 때문에 고질적 성능문제가 있음. [spring boot에서 JDK Dynamic proxy 사용 방법](https://multifrontgarden.tistory.com/282)
  - CGLIB(code generate library): 클래스의 바이트 코드를 조작하여 프록시 객체 생성. 인터페이스가 아닌 클래스도 생성 가능.
- spring AOP의 부가기능을 담은 advice가 적용되는 대상은 객체의 메서드. 독립적으로 개발한 부가기능 모듈을 다양한 객체의 메서드에 동적으로 적용시켜주기 위해 프록시를 사용한다. 스프링 AOP가 프록시 AOP라는 의미
- 물론, 프록시 방식이 아닌 **AspectJ**처럼 프록시방법을 쓰지 않고 부가기능을 제공하는 방법도 있다. 컴파일된 클래스 파일을 수정하거나 JVM에 로딩시에 바이트코드를 수정해서 부가기능을 제공한다

- AspectJ의 장점은 다음과 같다. (단점은 복잡하다는 것)
  - 바이트코드를 조작하기 때문에 스프링과 같은 DI 컨테이너의 도움이 필요 없다.
  - 프록시 방법을 사용하지 않기 때문에 클라이언트가 호출하는 메서드가 아니라 객체의 생성, 필드값 조회, static 변수 초기화 등과 같은 더 다양한 부가기능을 제공할 수 있다.
  - spring bean이 아닌 대상에도 AOP 가능

- AOP 용어
  - target: 부가 기능을 부여할 대상. 클래스 혹은 다른 프록시 객체 등
  - advice: target에 제공할 부가기능을 담은 모듈, 클래스, 객체, 메서드.
  - join point: advice가 적용될 위치. 스프링 프록시 AOP에서는 메서드의 실행 단계를 의미.
  - pointcut: advice를 적용할 join point를 선별하는 기능을 정의한 모듈. spring proxy aop에서 join point는 메서드의 시작이므로 포인트컷은 어드바이스가 적용될 메서드를 찾는 기능이라고 보면 된다.
  - advisor: pointcut과 advice를 하나씩 갖고 있는 object. spring aop에서만 사용하는 용어.
  - aspect: AOP의 기본 모듈. 하나 이상의 pointcut과 advice로 이루어짐. 보통은 singleton으로 존재. 어드바이저는 단순한 aspect


- - -

Spring AOP 사용 방법
=================

- spring aop를 사용하기 위해서는 dependency를 설정해야 한다

```
implementation("org.springframework:spring-aop")
```

- 메인 클래스에 아래와 같이 **@EnableAspectJAutoProxy**를 설정하자
- spring boot에서는 EnableAspectJAutoProxy를 설정할 필요 없다. **@SpringBootApplication**이 포함하고 있다.

```kt
@SpringBootApplication
//@EnableAspectJAutoProxy
class SpringdemoApplication
```

- aspect를 만들자. 클래스에 **@Aspect**을 추가하면 bean이 aspect로 동작한다. aspect안에 advice 메서드를 만들면 된다.

```kt
@Aspect
@Component
class AsepctExample {
    @Around("@annotation(myAopAnnotation)")
    fun add(joinPoint: ProceedingJoinPoint, myAopAnnotation: MyAop){
        if (myAopAnnotation.key.isEmpty()) {
            throw Exception("Empty")
        }
        joinPoint.proceed()
    }
}

@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class MyAop( //pointcut으로 사용될 annotation
    val key: String = "",
)
```

- 어드바이스가 동작되는 시간을 고를 수 있다. 이 때 원하는 pointcut을 지정가능
  - @Before: method 실행전에 작업 수행
  - @AfterReturning: method의 실행이 종료되어 값을 반환할 때
  - @Around: joinpoint의 동작 전후로 작업 수행.

- 위의 예제를 적용할 간단한 대상을 구현해보자

```kt
@RestController
class SampleController (private val sampleService: SampleService){
    @GetMapping("/a")
    suspend fun getA(): String{
        return sampleService.getA()
    }

    @GetMapping("/b")
    suspend fun getB(): String{
        return sampleService.getB()
    }
}

@Service
class SampleService{
    @MyAop
    fun getA(): String = "a"
    @MyAop("b")
    fun getB(): String = "b"
}
```

- 아래와 같은 명령어를 보내보면 AOP가 제대로 동작하는 것을 볼 수 있다

```
//에러 발생 X
curl -X GET "localhost:8080/b" -i
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8
Content-Length: 0

//에러 발생
curl -X GET "localhost:8080/a" -i
HTTP/1.1 500 Internal Server Error
Content-Type: application/json
Content-Length: 127
```
