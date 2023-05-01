---
layout: post
title:  "단위 테스트"
date:   2021-11-21 02:02:59
author: Inhyuk
category: test
tags: test
cover:  "/assets/instacode.png"
name: unit_test.md
---



단위 테스트
=========

- 단위 테스트는 단순히 테스트를 작성하는 것보다 더 큰 범주
- 단위 테스트에 시간을 들이는 만큼 그 이득을 최대화해야 함.

단위 테스트의 목표
----------

- 코드를 단위 테스트 하기 어렵다면 코드 개선이 필요하다는 의미: 강결합
- 단위테스트를 작성하지 않으면 처음에는 빠르게 시작할 수 있지만, 시간이 지날수록 점점 개발 속도가 느려진다.
- 코드베이스에서 변경은 무질서도를 증가시킴 -> 무질서도는 지속적인 리팩토링을 통해서 낮춰야함. 테스트는 리팩토링 시 안정망 역할을 해서 버그에 대한 보험을 제공할 수 있음.

좋은 테스트
--------

- 잘못된 테스트는 테스트가 없는 코드만큼 추후에 문제를 일으킬 수 있음.
- 테스트이 숫자가 많다고 테스트가 지향하는 목표를 이룰 수는 없다.
- 커버리지(분기, 코드) 지표는 부정 지표: 코드 커버리지가 적다는 것은 테스트가 충분하지 않다는 증거이지만, 코드 커버리지가 높다고 해서 test suite가 좋은 것이라고 할 수 없다.
  - test suite: 테스트 케이스의 묶음

성공적인 test suite
---------

- 모든 테스트는 개발 주기에 통합되어서 코드의 변경을 하면 테스트를 실행해야 한다.
- 단위 테스트는 시스템의 중요한 부분(비즈니스 로직, 도메인 모델)을 대상으로 한다.


정의
----

- 단위 테스트:
  - 작은 코드 조각을 검증
  - 빠르게 수행
  - 격리된 방식으로 자동화된 테스트

런던파 vs 고전파(디트로이트)
-----------------

- 단위 테스트에 대한 견해는 2가지로 나뉨
  - 런던파: 작은 코드 조각을 클래스와 메서드로 보고, 격리를 메서드내의 다른 협력자(다른 객체, 외부 의존성 등 변경 가능한 의존성) 대상으로 한다(test double 사용).
  - 고전파: 한 테스트에서 여러 클래스를 검증.

- 런던파 장점:
  - 한 메서드만 확인하기 때문에 테스트를 통한 버그 수정이 용이
  - 협력자들을 모두 test double로 대체하기 때문에 서로 연결된 클래스가 많고 복잡해져도 테스트가 복잡해지지 않는다
- 런던파 단점:
  - 테스트를 동작 단위보다 작은 단위로 테스트 하는 것은 의미가 없다
  - 복잡한 의존 관계 때문에 test double을 사용하는 것은 안티패턴이다. 오히려 리팩토링을 통해서 클래스 설계를 수정해야 한다
  - 과도한 명세 문제(over-specification): 테스트가 세부 구현과 밀접해지면서 리팩토링시 문제를 야기한다

- 의존성 종류
  - 공유 의존성: 테스트 간에 공유되는 의존성. 다른 app과 공유하는 DB, 정적 가변 필드(싱글턴) 등
  - 비공개 의존성: 공유하지 않는 의존성. VO, app에서만 사용하는 DB 등
  - 불변 의존성: 값 또는 값 객체
  - 프로세스 외부 의존성: app 외부에서 실행되는 의존성. 대부분은 공유 의존성(다른 api 서버, smtp 서버)이지만, app에서만 쓰이는 db는 비공개 의존성.

- 공유 의존성은 test double로 대체하는 것이 좋다.

||작은 코드|격리|test double 대상|
|-|-|-|-|
|런던|클래스,메서드|협력자|불변 의존성외 모든 의존성|
|고전|동작|다른 테스트|공유 의존성|

- 통합 테스트에 대한 견해
  - 런던파: 실제 협력자를 사용하는 모든 테스트를 통합 테스트로 간주
  - 고전파: 단위 테스트의 3개 조건 중 단 하나라도 위반하는 테스트를 통합 테스트로 간주.
    - 격리되지 않은 테스트: 공유 의존성을 사용하는 테스트는 다른 테스트들과 동시에 테스트 할 수 없기 때문에 격리되지 않은 테스트. 공유 DB 또는 다른 서비스에 대한 호출을 하는 테스트 등.
    - 둘 이상의 동작 단위를 검증: 비슷하고 느린 단계를 따르는 두 테스트를 하나로 묶은 테스트, 여러 모듈을 한 번에 테스트 하는 경우.

- end to end 테스트
  - 기본적으로 엔드투엔드 테스트는 통합 테스트의 일종
  - 엔드 투 엔드 테스트는 모든 의존성을 사용해서 최종 사용자의 입장에서 입력과 결과를 검증
  - 느리기 때문에 테스트 위치를 가장 마지막에 위치시켜 다른 테스트가 모두 성공한 경우에만 동작하게 하는 것도 좋다

AAA 패턴
------

- 단위 테스트는 가장 기본적으로 AAA(Arrange, Act, Asssert)패턴을 기초로 함
  - Arrange: SUT(테스트 대상 시스템)과 의존성의 상태를 설정
  - Act: SUT의 메서드 호출
  - Assert: SUT의 반환값, 상태 검증 및 협력자의 메서드 호출 여부 검증

- Given-When-Then: AAA와 유사.
- TDD에서는 실패할 테스트를 먼저 만들기 때문에 동작의 세부 구현을 알지 못한다. 동작에 대한 윤곽을 잡고 그 동작을 위해서 어떻게 개발 하면 되는지를 생각한다
- 단위테스트에서는 테스트에서 실행을 한 번만 하고, 통합 테스트에서는 실행을 여러 번 해도 괜찮다.
- 테스트내에서 조건문을 사용하는 것은 여러 테스트를 한 번에 한다는 것이므로 지양하도록 한다
- 보통 Act는 SUT의 한 메서드를 호출해서 이루어진다. 만약 여러 메서드를 호출한다면 캡슐화가 제대로 이루어지지 않았다는 증거이다
  - 불변 위반: 하나의 메서드 이후에 다음 메서드를 호출하지 않으면 모순이 생길 때. 캡슐화가 이루어지지 않았을 때 일어남

- 테스트 이름을 지을 때는 자유롭지만 명확하게 어떤 동작을 테스트하는지를 작성한다. 유틸리티 클래스가 아니면 메서드 이름을 포함하지 않는 것이 좋다(테스트는 메서드가 아닌 동작을 검증하는 것이기 때문)

- - -

단위 테스트의 4대 요소
===========

- 좋은 단위 테스트에는 4가지 요소가 존재
  - 회귀 방지(= 버그 방지)
  - 리팩토링 내성
  - 빠른 피드백: 테스트가 빨리 수행되어야 한다
  - 유지 보수성: 다른 사람과 미래의 작성자가 코드를 이해하기 쉬워야 하고, 테스트를 실행하기 어렵지 않아야 한다

버그 방지
-------

- 좋은 테스트는 버그를 방지한다
- 코드는 책임이기 때문에 개발이 진행될 수록 잠재적인 버그에 노출되는 경우가 늘어난다. 이러한 버그를 보호하는 테스트가 없으면 점점 잠재적인 버그가 쌓인다
- 버그는 테스트 중 실행되는 코드의 양이 클 수록, 코드가 복잡할 수록, 도메인 유의성이 높을 수록 발생할 확률이 증가한다. 그러므로 버그 방지를 위해서 이러한 코드를 테스트하는 것이 중요

리팩토링 내성
---------

- 리팩토링 내성: 테스트를 깨지 않으면서 코드를 리팩토링할 수 있는지. false positive(positive: 테스트가 실패했다는 경보. 실제 기능은 제대로 동작하지만 테스트가 실패를 반환)에 대한 내성
- 테스트에서 피해야할 false positive는 컴파일 오류를 내지 않는 실패이다. 매개변수가 하나 추가되거나 하는 false positive는 쉽게 해결할 수 있는 문제
- 테스트는 리팩토링시에 버그에 대한 지표가 되기 때문에 리팩토링을 안전하게 할 수 있게 해준다. false positive는 이점에서 문제가 발생함
  - 테스트가 리팩토링 내성이 없으면 테스트에 대한 신뢰가 떨어져서 테스트를 안전망으로 인식하지 않고 허위 경보라고 생각함(양치기 소년과 같음)
  - 테스트가 지속적으로 실패하기 때문에 코드 리팩토링 후에 테스트도 리팩토링 해야하고 이런 수고는 리팩토링에 대한 의지를 떨어뜨림

- false positive가 자주 일어나는 이유: 과도한 명세(테스트가 구현 세부사항에 결합도가 심함)
- 테스트는 해당 코드의 클라이언트(코드를 사용하는 존재)의 입장에서 테스트를 수행해야 한다. 해당 코드의 반환값, 최종 상태, 관리 외부 의존성 호출 여부만을 검증해야함

리팩토링 내성과 버그 방지
------------------

- 리팩토링 내성은 false positive(코드에 문제가 없지만 테스트가 실패함)를 위한 기준이고, 버그 방지는 false negative(코드에 문제가 있지만 테스트가 실패하지 않음)를 방지하기 위한 기준

||버그 없음|버그 있음|
|-|-|-|
|테스트 성공|좋음|false negative|
|테스트 실패|false positive|좋음|

- 보통 버그 방지를 위해 테스트를 작성하고 리팩토링 내성은 신경 쓰지 않지만 미래를 생각한다면 리팩토링 내성도 고려해야함. 테스트 코드 또한 부채임. false positive를 고려하지 않는 코드는 프로젝트가 커지면 커질 수록 문제를 일으키는 요인이 된다

이상적인 테스트
----------

- 이상적인 테스트(리팩토링 내성이 있고, 버그도 방지하면서 빠르고 다른 사람이 쉽게 이해할 수 있는 테스트)란 존재하지 않는다.

||버그 방지|리팩토링 내성|빠른 피드백|
|-|-|-|-|
|엔드투엔드 테스트|좋음|좋음|별로|
|간단한 테스트|별로|좋음|좋음|
|과도한 명세 테스트|좋음|별로|좋음|

- 엔드투엔드 테스트: 다른 서드파티도 같이 테스트하기 때문에 버그 방지가 우수하고, 최종 사용자 입장에서 테스트 하기 때문에 구현과의 결합도도 적어서 리팩토링 내성 우수. 느림
- 간단한 테스트: 빠르고, 리팩토링 용이. 버그 방지 기능 별로
- 깨지기 쉬운 테스트(과도한 명세): 버그 방지 용이, 빠름. 리팩토링 어려움

- 이상적인 테스트는 작성할 수 없고, 리팩토링 내성은 binary라서 포기할 수 없기 때문에 빠른 피드백과 회귀방지 중 하나를 선택해야 함.
  - 회귀 방지 완벽 - 엔드투엔드 테스트 - 통합 테스트 - 단위 테스트 - 빠른 피드백: 단위 테스트는 빠른 피드백을 강조하고, 엔드투엔드 테스트는 회귀방지를 강조, 통합테스트는 그 중간
- 최종 사용자의 동작과의 유사도와 테스트의 수는 반비례 관계이다; 최종 사용자의 동작과 유사한 엔드투엔드 테스트는 가장 적고, 유사하지 않은 단위 테스트의 수는 가장 많다
  - CRUD 기능만 있은 app은 엔드투엔드 테스트가 거의 없고, 비즈니스 복잡도가 적기 때문에 단위테스트의 수가 적어져서 통합테스트와 그 수가 같아진다

- - -

Mock
====

- test double은 논란의 주제. 런던파(test double을 적극적으로 사용해야 한다) vs 고전파(test double은 공유 의존성에만 사용)

Mock & Stub
------------

- test double: 의존성(공유 의존성, 비공유 의존성, 불변 의존성등)을 대체하는 가짜 의존성. 의존성을 대체해서 테스트를 편하게 해준다
- test double의 종류
  - mock: 외부로 나가는 상호 작용을 모방하고 **검사**. ex) SMTP 서버
    - mock: 프레임워크에 의해 자동으로 작성되는 mock
    - spy: 수동으로 작성하는 mock. Spring에서는 mock과 다르게 실제 동작과 의도된 동작을 지정할 수 있는 mock을 의미.
  - stub: 내부로 들어오는 상호 작용을 모방. ex) DB
    - stub: 시나리오마다 다른 값을 반환하는 의존성
    - dummy: null, 가짜 값이 하드 코딩된 값
    - fake: 존재하지 않는 의존성을 대체하는 stub
  - 많은 사람들이 위의 모든 것들을 mock이라고 부르기도 하지만 정확히는 위에서 설명한 하나만을 의미.
- 리팩토링 내성을 위해서 최종 결과물만을 검증해야 한다. stub호출은 최종결과물이 아니지만, mock 호출은 최종 결과물

```kotlin
@Test
fun `customer sends emails`(){
    //Mock
    val customer = Customer()
    val mock= mockk<EmailGateway>()

    customer.makeNewEmail(mock)

    //Mock으로 가는 호출은 최종 결과이기 때문에 호출을 검증한다
    verify(exactly = 1){ mock.sendEmail(any())}
}

@Test
fun `getting a customer by name`(){
    //Stub
    val testId = "test-id"
    val serivce = CustomerService()
    val mock= mockk<CustomerRepository>()

    val customer = service.getCustomer(testId)

    assertEquals(testId, customer.testId)
    //stub으로 가는 호출은 최종 결과가 아니기 때문에 검증을 하지 않는다.
    //stub 호출은 검증하는 것은 over-specification이다
}
```


- [CQS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation): 모든 메서드는 either 명령(command) or 조회(query)이어야 한다
  - command: 부작용을 초래. 보통은 반환값 없음. mock으로 대체.
  - query: 부작용 없음. 반환값 존재. [멱등성](https://ko.wikipedia.org/wiki/%EB%A9%B1%EB%93%B1%EB%B2%95%EC%B9%99)을 보장해야함. stub으로 대체
- CQS는 항상 따를 수 있는 것은 아님. 코드 특성 상 부작용을 초래하면서 값을 반환해야할 때가 있음.

식별과 공개
----------

- 공개/비공개 API: 클라이언트(코드를 사용하는 주체. 다른 코드, 최종 사용자 등)에서 접근할 수 있는/없는 API. ex) private, public
- 식별할 수 있는 동작 / 구현 세부 사항: 식별할 수 있는 동작은 클라이언트의 목표에 필요한 연산 혹은 상태를 노출하는 동작.

||공개 API|비공개 API|
|-|-|-|
|식별 가능 동작|좋음|거의 불가능|
|구현 세부 사항|지양|좋음|

- 이상적으로는 (공개 API,식별할 수 있는 동작), (비공개 API,구현 세부사항)으로 짝 지어야함
- 식별할 수 있는 동작은 비공개 API가 되는 것은 불가능. 그러나 구현 세부사항이 공개 API가 되는 경우가 존재하고 이는 피해야 한다.

```kotlin
class Customer {
    var name: String;

    constructor(name: String){
        this.name = name
    }

    fun normalizeName(name: String): String{
        //구현
    }
}

class CustomerClient {
    fun rename(id: Int, newName: String){
        val customer = customerRepository.findById(id)

        customer.name = customer.normalizeName(newName)

        customerRepository.save(customer)
    }
}
```

- 위 코드의 normalizeName은 식별할 수 있는 동작이 아니지만(클라이언트 입장에서 목표를 달성하는 데에 필요 없음), 공개 API로 되어있는 잘못된 설계이다.
- normalizeName은 Customer의 세부 사항이다

```kotlin
//구현 세부 사항을 비공개 API로 바꾼 코드
//kotlin에서 보통 이렇게 구현하지 않지만.....

class Customer {
    var name: String
    set(value) {
        field = normalizeName(value)
    }

    constructor(name: String){
        this.name = name
    }

    private fun normalizeName(name: String): String{
        //구현
    }
}

class CustomerClient {
    fun rename(id: Int, newName: String){
        val customer = customerRepository.findById(id)

        customer.name = newName

        customerRepository.save(customer)
    }
}
```

- 클라이언트에서 목표를 위해서 사용되는 클래스를 호출하는 횟수가 1보다 크면 구현 세부사항을 유출하고 있는 것이 아닌지 의심해봐야 한다
- 위에서 normalizeName에 포함될 내용은 불변성(모순이 되면 안되는 조건)이다. 공개/ 비공개를 잘 설정해서 캡슐화를 적용하면 불변성 위반을 방지할 수 있다. 캡슐화는 코드가 복잡해지면서 생길 수 있는 여러 부작용을 막는 유일한 수단

hexagonal architecture
-----------------

- hexagonal architecture: port, adaptor, service, domain 으로 구성된 아키텍처
  - port: 외부와 통신하는 입출구. ex) repository interface, controller에 사용되는 service등
  - adaptor: port에 연결되어서 실제로 외부와 통신하는 구현체. ex) controller, JPA repository
  - domain: 비즈니스 로직을 수행하는 부분
  - application service: 도메인 계층위에서 외부와 통신하는 곳. 비즈니스 usecase. (DB 조회 -> domain 동작 호출 -> DB 저장)가 기본
- 도메인 계층은 비즈니스 로직을 수행하고 외부 어플리케이션과 통신하고 DB와 상호작용하는 부분은 어플리케이션 서비스에 위임한다.
- 도메인 계층은 도메인들끼리 통신을 해야 하며 그 외의 클래스와 정보는 몰라야 한다
- 각 계층은 클라이언트를 위한 식별할 수 있는 동작을 제공해야 한다. 도메인의 식별할 수 있는 동작은 application service에서 사용하며, application service의 식별할 수 있는 동작은 클라이언트의 요구사항을 위한 것이다.

- hexagonal architecture의 세부 내용은 다른 곳에서 다루고 여기서는 test 관점에서 접근함

- app의 통신은 외부와의 통신과 내부에서의 통신(클래스 간의 통신)으로 나뉜다
- 내부 통신은 mock으로 하지않고, 외부와의 통신은 mock으로 처리해도 리팩토링시에 그대로 유지되기 때문에 리팩토링 내성을 갖는다
- 외부와의 통신을 모두 mock으로 하지 않고 공유 외부 의존성의 경우에만 mock으로 하고 db와 같은 비공유 외부 의존성은 그대로 사용한다
- 비공유 외부 의존성을 mock으로 대체하면 매개변수 타입 변경, 테이블 수정등에 의해서 리팩토링 내성이 떨어지는 것을 볼 수 있다

- - -

단위 테스트 방식
=======

- 단위테스트에는 3가지 방식이 존재
  - output based testing: 테스트 대상을 호출하고 나온 반환값을 검증. 함수형 테스트
  - state based testing: 테스트 대상이 작업을 한 후에 시스템의 상태(대상의 상태, 협력자의 상태, db의 상태 등)을 확인
  - communication based testing: mock과의 통신을 검증

- 각 테스트 방식은 리팩토링 내성에서 차이를 보인다. 리팩토링 내성 정도: 출력기반 테스트 > 상태기반 테스트 > 통신 기반 테스트
  - 출력 기반 테스트: 테스트 대상 메서드에만 결합하고 내부에는 관련되지 않으므로 리팩토링 내성 큼
  - 상태 기반 테스트: 시스템 내부 상태에 관심을 가지기 때문에 구현 내용과 결합일 될 가능성이 높다
  - 통신 기반 테스트: stub을 확인하는 경우가 있을수 있기 때문에 리팩토링 내성 가장 낮음


함수형 아키텍처
----------

- 메서드가 수학적 함수인지 확인하는 방법은 참조 투명성(메서드 동작을 값으로 하드코딩해도 차이가 없을때)을 확인하는 것
- 캡슐화가 되지 않아서 생기는 문제 중 가장 큰 부분은 불변성이 깨져서 모순이 생기는 것이다. 함수형 프로그래밍에서 생긴 객체는 불변이기 때문에 불변성이 깨질 이유가 없으므로 이러한 문제가 발생하지 않는다
- 함수형 아키텍처는 비즈니스 로직을 처리하는 코드와 부작용을 일으키는 코드를 분리하는 것. 부작용을 비즈니스 로직의 양 끝으로 몰아 넣는다
- 가별 셸(결정을 수행하는 코드)이 입력을 수집 -> 함수형 코어(결정을 내리는 코드. 수학적 함수)가 결정을 수행 -> 셸이 결정을 전달받아서 수행(부작용 발생)
- 가변 셸이 의사 결정을 추가하지 않게 결정(클래스)에 충분한 정보를 전달해야함. 가변셸은 아무 말 없이 수행에만 몰두해야 함.

- 함수형 코어는 출력 기반 테스트로 가변 셸은 통합테스트로 진행

- 함수형 아키텍처는 육각형 아키텍처의 일종이라고 볼 수 있다.
  - 차이점: 함수형 아키텍처에서는 모든 부작용을 가변 셸이 처리하고, 육각형 아키텍처에서는 부작용이 도메인 계층에서 일어날 수도 있다(도메인 계층에서 상태를 변경할 수 있다)
  - 함수형 아키텍처에서 가변셸은 application serivce에 의해서 도메인과 연결된다. 육각형 아키텍처에서는 이 둘을 묶어서 application service layer로 취급한다

- 함수형 아키텍처를 항상 적용 가능한 것은 아니다.
  - 예를 들어, 함수형 코어가 동작 중 특정 상황에 따라 DB에 대한 접근이 필요할 수가 있다. 이럴 때 함수형 코어에서 DB로 직접 접근하는 것은 좋지 못하다. 선택할 수 있는 경우는 1) 미리 DB에 접근해서 데이터를 갖고 전달한다. 2) 함수형 코어가 모든 결정을 담당한다는 조건을 완화해서 application layer가 함수형 코어의 상태를 확인해서 DB를 호출한다. 전자의 경우에는 필요없는 DB 접근이 이루어질 수 있다는 점이 단점으로 작용할 수 있고, 후자는 함수형 코어의 결정권한이 약해질 수 있다는 부작용이 있다.
  - 함수형 아키텍처는 기존의 아키텍처보다 성능적인 측면에서 좋지 않을 수가 있기 때문에 성능과 코드 유지보수성 사이에서 하나에 가중치를 주어서 선택해야한다
  - 도메인 모델을 불변으로 하는 것은 많은 비용이 들 수 있기 때문에 그 비용이 크면 함수형 아키텍처를 사용할 이유가 없다

- - -

리팩토링
======

- 코드는 2가지 기준으로 구별. 복잡도와 도메인 유의성 / 협력자 수
  - 복잡도: 코드 내 분기 수
  - 도메인 유의성: 도메인 문제에 의미 있는지 정도
  - 협력자 수: 가변 의존성 or 외부 의존성 수

||도메인 유의성, 복잡도 높음| 도메인 유의성, 복잡도 낮음 |
|-|-|-|
|협력자 수 많음|지양해야할 코드|application service(통합 테스트)|
|협력자 수 적음|도메인 모델(단위 테스트)|간단한 코드(테스트 X)|

- 복잡하면서 협력자도 많은 코드는 잘못된 코드이다. 험블 패턴을 이용한 리팩토링을 통해서 application service와 도메인 모델로 나눠야 한다
- humble pattern: 지나치게 복잡한 코드를 의존성과 로직으로 분할하고 그 것들을 얇은 humble wrapper로 감싼다. 가변셸, application service가 여기에 속한다. 분리를 하면 테스트를 쉽게 한다
- 험블패턴은 SRP와 연관되어 있다. 각 클래스는 단일 책임(로직을 수행하거나, 의존성 코드를 호출하거나)을 맡아야 한다는 원칙이다
- 코드의 깊이와 너비: 코드는 깊이가 깊거나(비즈니스 로직을 수행) 너비가 넓어야(연관된 의존성이 많다)한다

리팩토링 예제
--------

- 아래는 책의 내용을 kotlin으로 재구성한 것이다. 간략화 및 생략한 부분이 많다.

- 아래의 이메일 변경 서비스의 로직은 2가지다
  - 이메일이 회사 이메일이면 사용자를 Employee로 표기 아닐 시 Customer
  - 사용자가 유저에서 직원으로 혹은 그 반대로 변경된 경우에 대하여 회사 Employee 수를 관리


```kotlin
@Service
class UserService(
    val userRepository: UserRepository,
    val companyRepository: CompanyRepository,
    ){
    fun changeEmailById(userId: String, newEmail: String){
        val user = userRepository.getById(userId)

        if(user.id == newEmail) return

        val company = companyRepository.getAll().filter{it.domain == newEmail.split("@")[1]}


        val userType = if (company.isEmpty()) UserType.Employee else UserType.Customer
        val numDiffEmployee = if(userType == user.type) 1 else -1

        companyRepository.save(company[0].copy(number = company.number + numDiffEmployee))

        userRepository.save(user.copy(email = newEmail, type = userType))
    }
}

enum class UserType{
    Employee, Customer
}
```

- 위 코드는 복잡해보이지는 않지만 도메인 유의성이 높으면서도 협력자가 수가 많은 코드이다(지양해야할 코드).
- 가장 먼저 application service를 도입하고 도메인과 분할한다

```kotlin
@Service
class UserService(
    val userRepository: UserRepository,
    val companyRepository: CompanyRepository,
    ){
    fun changeEmailById(userId: String, newEmail: String){
        val user: User = userRepository.getById(userId)
        val company: Company = companyRepository.get()

        val numDiffEmployee = user.changeEmail(newEmail, company.domain)

        company.updateNumber(numDiffEmployee, newEmail)

        companyRepository.save(company)
        userRepository.save(user)
    }
}

data class User(
    val id: String,
    var type: Type,
    var email: String,
){
    enum class Type{
        Employee, Customer
    }

    fun changeEmail(newEmail: String, companyDomain: String): Int{
        if(email == newEmail) return 0

        val userType = if(companyDomain = newEmail.split("@")[1]) Type.Employee else Type.Customer

        email = newEmail

        return if(type!=userType){
            type = userType
            if(type == Type.Employee) 1 else -1
        } else 0
    }
}

data class Company(
    val domain: String,
    var numEmployee: Int,
) {
    fun updateNumber(numDiff: Int, email: String){
        if(domain == email.split("@")[1]){
            numEmployee += numDiff
        }
    }
}
```

- Domain 클래스인 *User*가 생겼고 여기서 *changeEmail*을 통해서 도메인 로직을 수행한다.
- *User*는 외부의존성이 없으므로 단위테스트하기 수월해졌다.
- 이 코드에도 문제점이 존재한다(책과 다르게 ORM, 의존성 주입 문제는 처음부터 생략했다)
  - user가 자신과 관련 없는 *numDiffEmployee*를 반환한다
  - 이전에 존재하던 **e-mail이 변경되면 update한다** 라는 로직이 사라졌다
  - email이 회사 이메일인지 확인하는 코드가 *User*에도 존재


```kotlin
@Service
class UserService(
    val userRepository: UserRepository,
    val companyRepository: CompanyRepository,
    ){
    fun changeEmailById(userId: String, newEmail: String){
        val user: User = userRepository.getById(userId)
        val company: Company = companyRepository.get()

        user.changeEmail(newEmail, company)

        companyRepository.save(company)
        userRepository.save(user)
    }
}

data class User(
    val id: String,
    var type: Type,
    var email: String,
) {
    enum class Type {
        Employee, Customer
    }

    fun changeEmail(newEmail: String, company: Company) {
        if (email == newEmail) return

        val userType = if (company.isEmailCorp(email)) Type.Employee else Type.Customer

        email = newEmail

        if (type != userType) {
            type = userType
            company.updateNumber(if (type == Type.Employee) 1 else -1)
        }
    }
}

data class Company(
    private val domain: String,
    private var numEmployee: Int,
) {
    fun updateNumber(numDiff: Int) {
        numEmployee += numDiff
    }

    fun isEmailCorp(email: String): Boolean{
        return domain == email.split("@")[1])
    }
}
```

- 수정 이후에도 도메인 코드들은 외부 의존성과 관련이 없다. 수정사항은 아래와 같다.
  - *User* 클래스에 새로운 협력자인 *Company*가 생겨서 *Company*에 대한 수정을 *User*가 담당한다
  - [TDA]() 원칙을 준수하기 위해서 *User*는 *Company*에 숫자를 업데이트 할 수 있는지를 묻지 않고 업데이트하라고 명령한다
  - *User* 클래스에서 e-mail이 회사 이메일인지 확인하는 로직을 *Company*에 부여한다

- 위의 코드는 함수형 아키텍처와 살짝 다르다. 도메인 로직을 수행할 때 어떤 부작용도 일으키지 않는 함수형 아키텍처와 달리, 도메인 로직에서 부작용(company, user의 수정)이 모두 도메인모델에 남아있다가 마지막 업데이트시에 DB로 그 부작용이 넘어간다

application service에서 로직 수행
----------------

- 위의 예제 코드들은 application service가 매우 간단했다. 데이터 검색 -> 비즈니스 로직 수행 -> 데이터 저장
- 실제로는 조건에 맞게 비즈니스 로직 수행 중에 데이터 검색이 이루어져야할 때가 있다. ex) 데이터 검색 -> 로직 1 수행 -> 데이터 검색 -> 로직 2 수행 -> 데이터 저장
- 이 문제를 해결하기 위해서는 3가지 방법이 있다. 3 방법 모두 도메인 격리/ service 간단/ 성능 사이에서 하나를 버리고 나머지를 선택하는 것이다
  - 가장자리에 위치: 모든 검색과 저장을 각 가장자리에 위치 시킨다. application service는 간단해지고 도메인과 외부 의존성 사이의 격리가 유지되지만 성능이 떨어진다
  - 도메인 모델이 외부 의존성 주입 받음: application service가 간단해지고 성능은 좋지만 외부 의존성과 도메인 사이의 결합이 생긴다
  - 의사 결정 프로세스 세분화: 중간 중간에 application serivce가 도메인 로직 수행에 필요한 데이터를 찾아서 주입한다. 도메인과 외부 의존성사이의 격리가 유지되고 성능은 좋지만 application service가 복잡해진다. 대부분이 이 선택을 한다
- 대부분의 동작은 성능이 중요하고 도메인에 의부 의존성을 주입하는 것은 지양해야 하는 것이므로 남는 것은 3번째 선택지이다. 3번째 선택지를 고르면 application service가 복잡해지기 때문에 이를 피할 방법을 생각해야 한다

CanExecute - Excute
-----------

- application service의 복잡도를 낮추는 하나의 방법은 application service에서 도메인 로직을 수행할 수 있는지를 직접 수행하는 것이 아니라 물어보는 것이다.

- 아래의 예제는 하나의 로직이 추가 되었다; *이메일이 확인된 경우에는 수정 불가능한다*

```kotlin
@Service
class UserService(
    val userRepository: UserRepository,
    val companyRepository: CompanyRepository,
    ){
    fun changeEmailById(userId: String, newEmail: String){
        val user: User = userRepository.getById(userId)
        val company: Company = companyRepository.get()

        user.changeEmail(newEmail, company)



        companyRepository.save(company)
        userRepository.save(user)
    }
}

data class User(
    val id: String,
    var type: Type,
    var email: String,
    val isEmailConfirmed: Boolean,
) {
    enum class Type {
        Employee, Customer
    }

    fun changeEmail(newEmail: String, company: Company) {
      if(user.isEmailConfirmed){
            throw Exception()
        }

        if (email == newEmail) return

        val userType = if (company.isEmailCorp(email)) Type.Employee else Type.Customer

        email = newEmail

        if (type != userType) {
            type = userType
            company.updateNumber(if (type == Type.Employee) 1 else -1)
        }
    }
}
```

- 위 코드는 1번 방법(가장자리에 위치)에 해당한다. 쓰지않을 수도 있는 *Company*을 받아서 저장해놓고 기다려야 한다. 이를 수정해보자.


```kotlin
@Service
class UserService(
    val userRepository: UserRepository,
    val companyRepository: CompanyRepository,
    ){
    fun changeEmailById(userId: String, newEmail: String){
        val user: User = userRepository.getById(userId)

        if(user.isEmailConfirmed){
            throw Exception()
        }

        val company: Company = companyRepository.get()

        user.changeEmail(newEmail, company)

        companyRepository.save(company)
        userRepository.save(user)
    }
}

data class User(
    val id: String,
    var type: Type,
    var email: String,
    val isEmailConfirmed: Boolean,
) {
    enum class Type {
        Employee, Customer
    }

    fun changeEmail(newEmail: String, company: Company) {
        if (email == newEmail) return

        val userType = if (company.isEmailCorp(email)) Type.Employee else Type.Customer

        email = newEmail

        if (type != userType) {
            type = userType
            company.updateNumber(if (type == Type.Employee) 1 else -1)
        }
    }
}
```

- 이번에는 *Company* 객체를 얻기 이전에 확인을 해서 불필요한 성능 저하를 피할 수 있었다. 그러나 문제는 *User*의 이메일이 업데이트 가능한지를 체크하는 로직이 application service에 남아있다는 것이다. 이는 캡슐화를 저해하는 설계다.

```kotlin
@Service
class UserService(
    val userRepository: UserRepository,
    val companyRepository: CompanyRepository,
    ){
    fun changeEmailById(userId: String, newEmail: String){
        val user: User = userRepository.getById(userId)

        if(!user.canChangeEmail()){
            throw Exception()
        }

        val company: Company = companyRepository.get()

        user.changeEmail(newEmail, company)

        companyRepository.save(company)
        userRepository.save(user)
    }
}

data class User(
    val id: String,
    var type: Type,
    var email: String,
    val isEmailConfirmed: Boolean,
) {
    enum class Type {
        Employee, Customer
    }

    fun canChangeEmail():Boolean{
      return !isEmailConfirmed
    }

    fun changeEmail(newEmail: String, company: Company) {
        if (!canChangeEmail()){
          throw Exception()
        }

        if (email == newEmail) return

        val userType = if (company.isEmailCorp(email)) Type.Employee else Type.Customer

        email = newEmail

        if (type != userType) {
            type = userType
            company.updateNumber(if (type == Type.Employee) 1 else -1)
        }
    }
}
```

- **CanExecute/Excute** 패턴을 적용하였다. 이 패턴을 적용하면 생기는 장점은
  - application service에서 *User*가 변경 가능한지에 대한 로직을 알 필요 없다
  - *User*에 변경 가능한지의 여부가 캡슐화 되어 있기 때문에, 로직이 변경되거나 추가시 수정이 용이하다

Domain Event
----------

- 도메인 이벤트: 외부 시스템에 통보하는 데 필요한 데이터가 포함되어 있는 클래스. application에서 domain의 변경 사항을 외부에 알리는 데에 사용되기도 함
- 위의 예제에서 *User*의 현재 이메일과 입력으로 들어온 이메일이 같으면 변경이 되지 않는다. 그렇기 때문에 email이 업데이트 되지 않아도 DB에 저장한다(의미 없는 행동). 서비스에 변경이 되었는지를 확인하는 로직을 직접 넣는 것은 복잡한 설계가 될 수 있다
- 책에서는 메세지 버스를 이용해서 도메인 이벤트의 필요성을 설명하였지만, 여기서는 간략화를 위해서 메시지 버스를 생략했었으므로 예시도 DB로 든다. 실제로 DB를 의미 없이 호출 하는 것은 성능에만 영향을 미치지만, 메세지 버스를 호출하는 것은 이 app의 식별할 수 있는 동작이고 변경이 없는 것을 외부와의 통신에 보내게 되면 불변성이 깨지게 된다.

```kotlin
@Service
class UserService(
    val userRepository: UserRepository,
    val companyRepository: CompanyRepository,
    ){
    fun changeEmailById(userId: String, newEmail: String){
        val user: User = userRepository.getById(userId)

        if(!user.canChangeEmail()){
            throw Exception()
        }

        val company: Company = companyRepository.get()

        user.changeEmail(newEmail, company)

        user.emailChangedEvents.forEach {
          companyRepository.save(company)
          userRepository.save(user)
        }
    }
}

data class User(
    val id: String,
    var type: Type,
    var email: String,
    val isEmailConfirmed: Boolean,
    val emailChangedEvents: MutableList<EmailChangedEvent> = listOf<EmailChangedEvent>
) {
    enum class Type {
        Employee, Customer
    }

    fun canChangeEmail():Boolean{
      return !isEmailConfirmed
    }

    fun changeEmail(newEmail: String, company: Company) {
        if (!canChangeEmail()){
          throw Exception()
        }

        if (email == newEmail) return

        val userType = if (company.isEmailCorp(email)) Type.Employee else Type.Customer

        email = newEmail

        if (type != userType) {
            type = userType
            company.updateNumber(if (type == Type.Employee) 1 else -1)
        }

        emailChangedEvents.add(EmailChangedEvent(id, email))
    }
}

data class EmailChangedEvent(
    val userId: String,
    val newEmail: String,
)
```

- domain event를 일반화해서 *DomainEvent* 클래스를 만들고 모든 클래스가 이 클래스의 list를 포함해서 사용할 수 도 있다. 이렇게 되면 event dispatcher를 구현하기 용이해진다
- 도메인 이벤트는 결정을 service가 아닌 도메인에 위치시키게 해준다

정리
---

- 결국 외부 시스템에 대한 app의 부작용을 최소로 하는 것이 좋다. 부작용을 최소로 하기 위해서는 부작용을 비즈니스 로직이 끝날때까지 DB 같은 다른 의존성이 아닌 메모리에 위치 시키면 된다. 그러면 외부 테스트 없이 단위테스트를 진행할 수 있다.
- 도메인 이벤트와 CanExecute/Execute 패턴을 이용해서 의사 결정을 최대한 도메인 모델에 위임하는 것이 좋다. 물론 항상 그런 것은 아니다. ex) 도메인 모델에서는 외부 의존성 없이 이메일의 유일성을 검증할 수 없다. 프로세스 외부 의존성에 기반한 도메인 로직의 분기가 있을 경우에는 도메인이 모든 것을 결정할 수 없음
- service에서 모든 로직을 제거할 수 없듯 도메인에서 협력자를 제거하지 못하더라도 외부 의존성만 아니면 괜찮다. 그렇더라도 이런 협력자는 외부 공유 의존성이 아니면 **mock을 사용하면 안된다**
- 검증은 클라이언트의 목표와 관련되어 있는 메서드 or 외부 공유 의존성인 경우에만 확인해야 한다. 이 둘은 식별할 수 있는 동작이다.
- 모든 테스트는 현재 클라이언트(클래스, 사용자 등)의 입장에서 수행해야 한다. 그 밑의 계층에는 관심을 갖지 말아야 한다

- - -

통합 테스트
=========

- 단위 테스트만 사용하면 의존성에 대한 테스트가 진행되지 않기 때문에 시스템이 제대로 동작하는지 알 수 없을 수 가 있다.


통합 테스트 구조
-----------

- 통합 테스트는 단위 테스트의 3 가지 요소(단위 코드 조각 테스트, 격리된 상태, 빠르게) 중 하나를 만족하지 않는 테스트. 보통은 외부 의존성을 통합해 테스트 해보는 것을 의미
- 상술했듯이 단위 테스트는 도메인 로직과 알고리즘에 적용하고 application service는 통합테스트를 진행한다
  - application service 테스트의 외부 의존성을 mock으로 대체하면 단위 테스타가 될 수 있지만, mock으로 대체 못하는 경우들도 있다. 이런 경우에는 통합 테스트를 진행해야 한다

- 통합테스트는 느리고 유지비가 많이 든다. 그러나 버그 방지에는 우수한 성능을 보인다.
- 통합 테스트는 비즈니스 시나리오의 예외 상황 및 흐릅을 확인하는 테스트이므로 가장 긴 흐름을 테스트 하는 것이 중요한다

외부 의존성
-----

- 통합 테스트는 위부 의존성을 거치는 흐름을 테스트 해야 한다. 이때 외부 의존성은 2가지로 나뉜다
  - 관리 의존성: 테스트하려는 app 외에는 접근할 수 없고 오직 이 app을 통해서만 접근 가능한 의존성. ex) DB
  - 비관리 의존성: 외부에서 이 app과 의존성 사이의 상호작용을 확인할 수 있는 의존성. ex) SMTP 서버

- 보통은 **관리 의존성은 실제로, 비관리 의존성은 Mock으로 테스트**해야한다.
  - vs 엔드투엔드 테스트(모든 의존성을 전부 실제 인스턴스로 사용하여 테스트). 관리 의존성만을 mock으로 한 통합 테스트의 버그 방지 수준은 엔드투엔드 테스트와 비슷하지만, 몇 테스트는 엔드투엔드 테스트로서의 기능(관리 의존성도 테스트)으로써의 역할을 하도록 남겨 놓는 것도 좋은 선택

- 가끔은 관리이면서 비관리인 의존성이 존재한다. ex) 다른 app에서 접근 가능한 DB
  - 이런 경우에는 테이블별로 확인해야 한다. 다른 app과 같이 사용하는 테이블은 비관리 의존성으로, 다른 app이 사용하지 않는 테이블은 관리 의존성으로 생각한다.

- 관리 의존성을 mock으로 대체하는 것은 리팩토링을 힘들게 하고, 통합테스트의 버그 방지로서의 기능도 떨어지게 한다. DB는 최대한 같은 형태의 DB를 사용하도록 하자

인터페이스
-------

- 여러 테스트나 코드를 보다보면 인터페이스를 남용하는 경향이 있다
- 인터페이스는 Dependency inversion을 위해서 사용한다. 이렇게 되면 사용자 코드를 고치지 않고서도 새로운 기능을 구현할 수 있는 OCP 원칙을 따를 수 있다. 하지만 이것은 구현이 2가지 있을 수 있을 때만 가능한 것이다.
- 두 개 이상의 구현체가 사용하지 않은 interface을 만드는 것은 YAGNI 원칙에 위배되는 것이다. 언제 사용할지도 모르는 코드를 미리 작성하는 것은 불필요한 의미 없는 비용을 계속해서 지불하는 것이다

#### OCP vs YAGNI

- [OCP VS YAGNI](https://enterprisecraftsmanship.com/posts/ocp-vs-yagni/)
- OCP: open closed principle. 2 가지 버전이 존재
  - Bertrand Meyer의 버전: close의 대상이 외부 사용자
  - Bob Martin의 버전: 기존의 코드를 수정하지 않고 변화하는 것에 대응할 수 있어야함. close의 대상이 기존의 코드
- YAGNI: 현재 필요하지 않는 기능을 더 넣을 필요는 없다. Bob Martin의 방식과 충돌
- YAGNI vs Bob Martins' OCP: APP과 같이 코드가 최종이면 YAGNI가 좋고, library나 framework의 경우에는 Bob Martin의 OCP를 따라야 한다

사례
----

- 좋은 통합 테스트를 위해서는 몇 가지 방법이 있다
  - 도메인 모델과의 경계 명시: 도메인 모델을 코드에서 잘 보이도록 위치 시켜놓아야 한다. 경계가 명확하면 테스트시 통합테슽와 단위 테스트의 차이를 보여준다
  - app내의 layer 수 줄이기:
    - layer가 너무 많으면 코드 베이스에서 로직을 이해하기 어려워 진다.
    - 추상화가 지나치게 많으면 테스트가 어렵다.
    - 계층이 너무 많으면 도메인 모델과 service 사이의 명확한 경계가 없어지고 각 계층을 따로 테스트 하다보니 통합 테스트가 의미 없어진다
    - 대부분의 BE 서버는 도메인 모델, application layer, infrastructure layer(도메인에 속하지 않는 알고리즘, 프로세스 외부 의존성에 접근하는 코드) 이 3가지만 있으면 된다
  - circular dependency 제거: 코드를 읽고 이해하기 어렵게 만든다. 테스트를 할 때 mock을 사용하지 않으면 테스트가 힘들게 된다.

- - -

Mock과 통합 테스트
=========

- *Mock*은 모방 + 검증. 비관리 의존성에 대해서만 적용해야 함.
- Mock을 비관리 의존성에 사용할 때는 app의 가장 경계에서 사용해야 한다. 가장 경계에서 해야 가장 많은 양의 코드를 테스트해서 버그 방지를 더 높일 수 있다. 또한, 리팩토링 내성도 향상된다.
- mock은 꼭 통합 테스트에만 적용해야 한다. 기본적으로 비즈니스 로직과 orchestration은 분리해야 한다.
- mock을 사용하는 경우에는 호출을 검증해야 한다. 호출한 parameter와 횟수 등등
- 서드파티 타입은 직접 mock으로 만들지 말고, 래퍼로 감싼 후에 mock처리를 해야 한다. 서드파티 코드에 대한 동작을 잘 모를 수 있기 때문이다.

- - -

DB 테스트
=======

-

- - -


안티 패턴
=====

- 단위 테스트의 안티 패턴은 좋아 보이지만 미래에는 문제를 일으켜서 피해야할 패턴들이다

비공개 메서드 테스트
---------

- 비공개 메서드는 테스트 하지 말아야 한다
- 비공개 메서드는 식별할 수 있는 대상이 아니다. 비공개 메서드를 테스트 하는 것은 over specification test이다
- 비공개 메서드를 테스트 해야한다는 의미는 추상화가 부족하다는 의미이고 별도의 클래스를 만들어야 한다는 뜻이다

비공개 상태 테스트
-------

- 비공개 메서드를 테스트하는 것 만큼 private한 상태를 테스트하거나 노출하는 것은 안티패턴의 한 종류 이다
- 비공개 상태를 테스트 하지말고 클래스에서 이 상태를 사용하는 방법을 알아보고 그 방식으로 테슽 해야 한다


테스트로 구현 유출
-------

- 도메인 지식을 테스트로 유출하는 것도 안티패턴이다
- 복잡한 알고리즘을 테스트하다보면 종종 있는 경우

```kotlin
class Calculator{
    fun getGcd(num1: Int, num2: Int):Int{
      var a = num1
      var b = num2

      if(num1>num2){
          b = num1
          a = num2
      }

      while (a!=0){
          val tmp = b%a
          b = a
          a = tmp
      }

      return b    
    }
}

class CalculatorTest{
    @Test
    fun `gets GCD of two numbers`(){
        val calculator = Calculator()
        var a = 12
        var b = 27

        while (a!=0){
            val tmp = b%a
            b = a
            a = tmp
        }

        assertEquals(b, calculator.getGcd(a,b))
    }
}
```

- 위의 예제는 최대공약수를 구하는 로직이다. 테스트에 어떻게 테스트를 gcd를 구하는 지에 대한 로직이 노출되어 있다.
- 아래와 같이 값을 하드 코딩하는 것이 좋다


```kotlin
class Calculator{
    fun getGcd(num1: Int, num2: Int):Int{
      var a = num1
      var b = num2

      if(num1>num2){
          b = num1
          a = num2
      }

      while (a!=0){
          val tmp = b%a
          b = a
          a = tmp
      }

      return b    
    }
}

class CalculatorTest{
    @Test
    fun `gets GCD of two numbers`(){
        val calculator = Calculator()
        val a = 12
        val b = 27

        assertEquals(3, calculator.getGcd(a,b))
    }
}
```

테스트에만 필요한 코드 추가
-------------

- 테스트에만 필요한 코드를 추가하는 행동은 하지 말아야 한다
