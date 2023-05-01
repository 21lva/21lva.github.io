---
layout: post
title:  "클린아키텍처 as Kotlin"
date:   2022-04-25 02:02:59
author: Inhyuk
category: architecture
tags: test
cover:  "/assets/instacode.png"
name: archi_java.md
---

layered architecture의 문제점
============

- 계층형 아키텍처의 기본 구조: 웹 -> 도메인 -> 영속성(persistent)

계층형 아키텍처는 DB 주도 설계를 하도록 유도
------

- 계층형 아키텍처의 토대는 DB. 나머지 계층은 도메인이 영속성(DB)에 의존하고 영속성이 도메인 계층에 의존하게 되는 구조.
- 어플리케이션에서 가장 중요한 것은 도메인 모델이기 때문에 다른 요소들이 도메인 모델에 의존하는 구조가 되어야 한다.
- DB 중심적인 아키텍처가 형성되는 데에 큰 역할을 한 것이 **ORM**이다. ORM의 **Entity**들은 도메인 계층의 서비스가 사용하며 보통 영속성 계층에 위치한다. 즉, 도메인 계층이 영속성 계층에 의존하게 된다. 이러한 엔티티를 도메인 모델로 그대로 사용하는 경우도 발생하며 이는 영속성계층에서 처리해야할 loading, db transaction, flush등의 작업들이 도메인 계층에도 영향을 미치게 된다.

```
도메인 계층  | 영속성 계층
[service]  =====> [repository]
             \          |
              \         v
               \---> [entity]
```

- 위와 같은 영속성 계층을 계속 유지하면 영속성 계층으로 도메인 관련 로직들과 유틸리티 등이 내려갈 수 있다.

테스트 하기 어려움
-------

- 계층형 구조를 유지하면 계층을 건너뛰는 코드를 작성하기 마련이다. 이 방법은 2가지 문제점을 만든다
  1. 도메인 로직이 웹계층에 침범할 수도 있다.
  2. 웹 계층 테스트에서 도메인 계층과 영속성 계층을 mocking해야 할 수도 있다. 이는 테스트 하기 어려운 구조를 만든다.


유스케이스를 숨긴다
--------------

- 계층형 구조에서는 도메인 로직이 여러 계층으로 뿌려진다. 유스케이스를 추가하거나 변경할 때 이러한 도메인 구조를 찾는 것이 어려워진다.
- 시간이 점점 지나면 하나의 서비스가 여러 객체들(웹, 영속성의 객체들)에 의존하게 되면서 서비스가 비대해지게 된다. 이는 응집력을 낮추게 된다.

- - -

의존성 역전하기
=============

SRP
---

- SRP의 뜻은 *하나의 컴포넌트를* **변경***하는 이유는 하나여야 한다*는 것이다. 하나의 컴포넌트가 하나의 이유로 변경된다면 다른 이유로 소프트웨어가 변경될 때 이 컴포넌트에 대해 변경을 할 이유가 없어진다.

의존성 역전 원칙
-----------

- 계층형 아키텍처에서의 의존성 방향은 아래로(웹->도메인->영속성)으로 향한다. 대부분의 코드는 SRP를 만족하지 않기 때문에 하위 계층의 변경은 상위 계층으로 전파된다. 이러한 구조는 가장 중요한 부분인 도메인 코드가 로직이 아닌 다른 부분으로부터 영향을 받는 문제를 야기한다.
- 의존성 역전 원칙을 이용하여 도메인 영역->영속성 구조를 **영속성 계층->도메인 계층(인터페이스<-도메인 서비스)**으로 의존성을 바꾸면 도메인 계층이 영속성 계층으로부터 받는 영향을 최소화 할 수 있다.

```
도메인 계층                      |       영속성 계층
서비스 -> repository(interface) <- repsitory(구현체) -> ORM 엔티티
 |                            |
 v                            |
 도메인 entity                  |
```

클린 아키텍처 & 헥사코날 아키텍처
------

- 클린 아키텍처는 계층들이 양파처럼 도메인 계층을 감싸는 형태이다.
  - 코어: 도메인 엔티티, 도메인 서비스들이 존대. 도메인 코드들이 영속성과 웹계층(UI)등에 의존하지 않기 때문에 비즈니스 로직에 집중할 수 있다.
- 클린 아키텍처는 비용이 추가된다. e.g. 영속성 계층에서 ORM을 사용하는 경우에 특별한 엔티티 클래스가 필요로 하게 된다. 도메인 계층에서는 이러한 엔티티에 의존하지 않기 때문에 영속성 계층에서 자체적으로 도메인 계층의 엔티티를 저장하기 위한 엔티티를 따로 만들어야 한다.

- hexagonal architecture는 조금 더 구체화된 구조를 가진다. (내가 찾아봤을 때는 헥사고날=클린아키텍처 + 포트 어댑터이다. 뭐 사실 크게 다르지는 않다)
- 육각형 아키텍처의 구조는 아래와 같다.

```
=> : 구현
-> : 포함
인바운드 어댑터(UI) -> 입력 포트 <= 유스케이스 -> 아웃바운드 포트 <= 아웃바운드 어댑터
                                  |
                                  V
                                엔티티
```

- 어플리케이션의 코어(유스케이스, 엔티티)와 외부(어댑터)가 통신하려면 코어가 어댑터들에게 포트를 제공해야 한다.
  - driving(in-bound) port: 유스케이스의 인터페이스. 유스케이스가 해당 인터페이스를 구현한다
  - driven(out-bound) port: 유스케이스가 포함하는 인터페이스. 영속성에서 해당 인터페이스를 구현한다.

- - -

코드
===

- 보통 계층형 아키텍처의 패키지 구조는 다음과 같을 것이다. 아래의 코드는 도메인 계층과 영속성 계층을 의존성 역전하였다.

```
root
|-- domain
|   |- Account
|   |- Activity
|   |- AccountRepository
|   |- AccountService
|
|-- persistence
|   |- AccountRepositoryImpl
|
|-- web
    |- AccountController
```

- 이 코드는 몇 가지 문제가 있다.  
  1. app이 기능적으로 분해되지 않았다: 현재의 구조에서 다른 기능을 추가한다면 web, persistence, domain 패키지에 각각 코드를 추가해야 한다. 이는 서로 연관 없는 기능끼리 뭉치게 하는 단점이 있다
  2. usecase가 전혀 파악되지 않는다: AccountService는 어떤 usecase를 구현한지 알 수 없다.
  3. 패키지 구조를 통해서 아키텍처의 목표를 알 수 없다.

- 문제들을 해결하기 위하여 패키지를 기능적으로 구성해보자

```
account
|- Account
|- Activity
|- AccountRepository
|- SendMoneyService
|- AccountController
|- AccountRepositoryImpl
```

- 기능적으로 코드를 묶기 위해서 account 패키지로 묶었다. Service의 기능을 보여주기 위해서 클래스 명도 변경하였다(screaming architecture)

- 이 방식이 최선일까? 이 방식도 또 다른 문제점을 갖고 있다
  1. 기능적인 패키징은 계층형보다 가시성을 떨어뜨린다
  2. 도메인 코드가 다른 영역의 코드에 접근하는 것을 막을 수 없다

- 다시 한 번 구조를 변경해보자

```
root
|--account
    |-- adaptor
    |   |- in
    |   |   |- web
    |   |       |- AccountController
    |   |       
    |   |- out
    |       |- persistence
    |           |- AccountPersistenceAdaptor
    |           |- SpringDataAccountRepository
    |          
    |-- domain
    |   |- Activity
    |   |- Account
    |
    |-- applicaation
        |- SendMoneyService
        |- port
            |- in
            |   |- SendMoneyUsecase
            |
            |- out
                |- LoadAccountPort
                |- UpdateAccountStatePort
```

- 육각형 아키텍처의 핵심 요소인 **엔티티, 유스케이스, 어댑터, 포트**가 포함된 구조이다.
- 이 구조의 장점은 기능적 패키지와 구조적 패키지의 장점을 얻으면서 둘의 문제점은 해결할 수 있다는 것이다.
- 패키지에 들어 있는 모든 클래스들은 application 패키지 내에 있는 port를 통하지 않고서는 바깥에서 사용하지 못하도록 package-private 접근으로 해도 된다. 물론, port는 adaptor에서 접근해야 하므로 public이어야 한다. 도메인 코드들은 서비스에서 접근해야 하므로 public이어야한다. 서비스는 usecase(in-bound port)를 거쳐서 접근해야 하므로 public일 필요가 없다.
- 어댑터 package는 같은 port에 대한 여러 다른 어댑터를 가질 수 있다.
- 이 코드는 DDD의 개념을 적용하기 쉽게 해준다. 위의 account package는 bound context의 역할을 할 수 있다.

의존성 주입
--------

- 육각형 아키텍처에서 가장 중요한 것 중 하나는 내부의 요소들(포트, 유스케이스, 서비스, 엔티티)이 어댑터에 의존하지 말아야한다는 사실이다.
- in-bound 어댑터는 동작 방향이 의도한 방향이기 때문에 괜찮지만 out-bound 어댑터는 동작 방향과 의도한 방향이 다르기 때문에 의존성 역전을 해주어야 한다. 위에서 살펴본 방식으로 처리하면 된다. application 계층에 port용 인터페이스를 만들고 이 인터페이스를 어댑터에서 구현하는 형태가 되면 된다.
- 이 때 포트인터페이스에 실제 어댑터를 매핑시켜주는 역할은 **의존성 주입**이 맡는다.

- - -

Usecase
=========

- 육각형 아키텍처에서 어플리케이션 서비스는 2 가지로 이루어진다: 유스케이스(인터페이스), 서비스. 서비스는 엔티티를 오케스트레이션하는 역할을 한다.
- 먼저 도메인 모델을 구현하자

도메인
----

todo

유스케이스
----

- 유스케이스가 하는 일은 입력 -> 비즈니스 규칙 검증 -> 모델 조작 -> 출력 반환
- 유스케이스에서는 **비즈니스 규칙 검증**의 책임을 맡는다.
- 입력 유효성 검증 vs 비즈니스 규칙 검증: 검증을 위해 도메인 모델이 필요한 경우에는 비즈니스 규칙 검증이고 그렇지 않은 경우에는 입력 유효성 검증이다.
- 넓은 서비스 문제를 피하기 위해서 모든 유스케이스를 한 서비스 클래스에 넣지 않는다.

```kotlin
//service 구현체

package com.example.cleanarchitecture.account.application.service

import com.example.cleanarchitecture.account.application.SendMoneyUseCase

class SendMoneyService: SendMoneyUseCase{
    private val loadAccountPort: LoadAccountPort
    private val accountLock: AccountLock
    private val updateAccountStatePort: UpdateAccountStatePort

    override fun sendMoney(command: SendMoneyCommand): Boolean {
        // 비즈니스 규칙 검증
        // 모델 상태 조작
        // 출력 값 변환
    }
}
```

입력 유효성 검증
------------

- 입력 유효성은 서비스 클래스의 책임은 아니지만 어플리케이션 계층의 역할이다. 어플리케이션 영역에서 포트로부터 전달 받은 DTO의 유효성을 먼저 검증해야 도메인 로직을 수행할 수 있기 때문이다. 이 때 입력모델(DTO)에서 직접 검증을 하도록 할 수 있다.

```kt
data class SendMoneyCommand(private val accountId: AccountId, private val targetAccountId: AccountId, private val money: Money): SelfValidating<SendMoneyCommand>() {
    init {
        validateSelf()
    }

    override fun validateSelf() {
        TODO("Not yet implemented")
    }
}

abstract class SelfValidating<T>{
  //@NotNull등을 이용한 구현체로 만들 수도 있다.
    abstract fun validateSelf()
}
```

유스케이스와 입력 모델
--------------

- 유스케이스마다 다른 입력 모델을 사용하는 것이 좋다. 유스케이스 전용 입력 모델은 유스케이스을 명확하게 표현하고 추후에 유스케이스가 변경될 때 다른 유스케이스가 이것으로부터 영향을 받는 것을 최소화 할 수 있다. 한 가지 안타까운 점은 각 입력 모델을 만드는 수고를 들여야한다는 것이다.

비즈니스 규칙 검증
----------------

- 비즈니스 규칙 검증은 유스케이스의 로직의 일부이다. 입력 유효성을 검증하는 것은 **syntatic**이고, 비즈니스 규칙 검증은 **semantic**이라고 할 수 있다.
- 비즈니스 규칙 검증과 유효성 검증의 차이는 현재 도메인 모델의 값이 필요한지 아닌지에 달려 있다. 예를 들어, "출금은 계좌의 잔금을 초과할 수 없다"라는 규칙을 검증하기 위해서는 현재의 계좌 잔금이 필요하다. 그렇기 때문에 이 규칙을 검증하는 것은 비즈니스 규칙 검증에 속한다. BUT "출금 금액은 0보다 커야 한다"는 규칙은 도메인 모델이 필요 없기 때문에 유효성 검증에 속한다.
- 비즈니스 규칙을 검증하는 방법은 여러가지가 있다
  1. 엔티티에 내제한다
  2. 유스케이스 코드에서 직접 검증: 엔티티에서 규칙을 검증하기 힘든 경우에 사용

anemic domain model vs rich domain model
-----------------------

- 여기서 말하는 아키텍처는 두 도메인 모델 중 어떤 것을 고르더라도 적용할 수 있는 구조이다.
- rich 도메인 모델은 도메인 자체에 최대한 도메인 로직을 구성하고 유스케이스는 단지 이 모델에 위임하는 역할을 하고, anemic 도메인 모델을 사용하면 도메인 로직을 유스케이스에서 처리하고 모델은 단지 값을 갖고 있는 형태가 된다.

유스케이스 마다 다른 출력 모델
----------------------

- 위에서 유스케이스마다 다른 입력 모델을 사용하는 것이 좋다고 하였다. 출력 모델도 그렇다. 출력은 호출자에게 꼭 필요한 데이터만을 전달해야한다. 다른 유스케이스와 같은 출력 모델을 쓰는 경우에는 필요 없는 데이터를 전달하는 경우가 발생할 수 있다. 이러한 이유는 도메인 엔티티를 출력 모델로 사용하면 안된다는 주장에도 사용될 수 있다.

읽기 전용 유스케이스
----------

- 읽기 전용 유스케이스는 다른 유스케이스와 분리되어서 사용할 수 있게 하기 위해서 **Query**로 명명할 수 있다.

```kt
class GetAccountBalanceService: GetAccountBalanceQuery{
    override fun getBalance(accountId: AccountId): Money {
        return loadAccountPort.loadAccount(accountId).calculateBalance()
    }
}
```
- 쿼리 서비스는 유스케이스 서비스와 비슷하게 동작한다. 그렇더라도 코드로 명확하게 구분해야한다.

- - -

웹 어댑터
=======

- hexagonal architecture에서 모든 외부와의 통신은 **adaptor**가 담당한다
- 어댑터는 2가지 종류가 있다: driving, drivien. 이 중 웹 어댑터는 driving에 속한다. 외부로부터 들어오는 요청을 어플리케이션에 전달하는 역할을 수행한다.
- 어플리케이션 계층은 어댑터를 위한 포트를 제공한다. 웹 어댑터는 이 포트를 호출하는 방식으로 동작한다


- 웹 어댑터는 다음과 같은 역할을 수행한다.
  1. HTTP Request를 객체로 매핑
  2. 권한 검사
  3. Request 유효성 검증: uscease의 입력 모델은 유스케이스의 맥락에서 유효한 입력을 검증하고, 여기서는 다른 검증을 수행한다.
  4. 입력을 usecase의 input model로 매핑
  5. usecase 호출
  6. usecase의 출력을 HTTP Response로 매핑 후 반환

컨트롤러 나누기
----------

- 컨트롤러를 구성할때는 하나의 usecase에 하나의 컨트롤러를 만드는 정도로 클래스를 작게 만들어야한다. 테스트할 때에도 컨트롤러에 코드가 많으면 테스트하기 힘들어진다. 가장 중요한 이유는 모든 연산을 단일 컨트롤러에 넣으면 input model와 같은 데이터의 재활용을 하도록 유횩한다. 이는 위에서 설명했듯이 지양해야할 방식이다.
- 아래는 적잘하게 만든 컨트롤러이다.

```
@RestController
class SendMoneyController(private val sendMoneyUseCase: SendMoneyUseCase){
    @PostMapping("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}")
    fun sendMoney(@PathVariable("sourceAccountId") sourceAccountId: Long, @PathVariable("targetAccountId") targetAccountId: Long, @PathVariable("amount") amount: Long){
        SendMoneyCommand(sourceAccountId, targetAccountId, amount).also {
            sendMoneyUseCase.sendMoney(it)
        }
    }
}
```

- - -

영속성 어댑터
==========

- 영속성 어댑터는 의존성 주입을 적절히 사용해야 데이터베이스 주도 설계를 피할 수 있다. 영속성 어댑터가 구현하는 포트는 서비스 계층에 있다.
- 영속성 어댑터가 하는 일은 다음과 같다
  1. 입력을 받는다
  2. 입력을 DB에서 사용하는 형식으로 변경한다. e.g. 도메인 모델을 ORM의 entity로 변경
  3. 입력을 DB로 전달 후 결과를 전달 받아서 어플리케이션 포맷으로 변경
  4. 출력을 반환

- 영속성 어댑터에 전달하는 DTO는 영속성 어댑터가 아닌 어플리케이션 계층에 구현되어 있다. 즉, 영속성 어댑터가 코어에 영향을 미치는 것을 최소화 해준다.

포트 인터페이스 분할
-------------

- 영속성 포트와 어댑터를 구현할 때 가장 중요한 것은 포트 인터페이스를 어떻게 나눌 것인가에 대한 고민이다.
- 대부분의 경우에는 하나의 어댑터에 하나의 포트라는 설계를 선호한다. 이렇게 되면 어플리케이션의 서비스가 포트내의 하나의 메서드만을 사용하더라도 필요 없는 포트 인터페이스에 의존하게 된다.
- ISP(인터페이스 분리 원칙): 넓은 인터페이스를 지양하고 인터페이스를 여러 개로 분리해서 필요한 인터페이스에만 의존하게 만드는 원칙. 아래는 이 원칙을 지켜서 만든 구조이다.

```
Service layer        | Persistence
Service1 -           |  Adaptor
         | -> Port1 <=====|
         | -> Port2 <=====|
                          |
Service2 -                |
         | -> Port3 <=====|
```

영속성 어댑터 분할
-------------

- 위의 구조는 하나의 영속성 어댑터로 다수의 영속성 포트를 구현하는 설계이다.
- DDD를 따르는 구조에서는 하나의 aggregate에 하나의 어댑터를 생성하는 방식으로 구현한다. 어그리케이트당 하나의 어댑터는 추후에 bounded context 분할을 위한 좋은 조건이 된다.
- 영속성 어댑터를 더 많은 클래스로 나눌 수 있다. 도메인 코드는 어떤 영속성 어댑터가 어떤 영속성 포트를 구현하는지에 대해 전혀 모른다.

```
interface AccountRepository: JpaRepository<AccountJpaEntity, Long>

interface ActivityRepository: JpaRepository<ActivityJpaEntity, Long>

class AccountPersistenceAdapter(private val accountRepository: AccountRepository, private val activityRepository: ActivityRepository): LongAccountPort, UpdateAccountStatePort{
  //생략

  override void updateActivities(account: Account){//Account를 AccountJpaEntity로 매핑하는 방식을 사용하는 것이 좋다. 그렇지 않으면 도메인 모델에 JPA annotation을 사용해야 하는 문제가 발생한다.
    //생략
  }
}
```

트랜잭션
-----

- transaction은 하나의 특정한 유스케이스에 대해서 일어난 모든 쓰기 연산을 처리해야 한다. 영속성 어댑터는 유스케이스에서 어떤 다른 어댑터들이 같이 동작하는지 알지 못하기 때문에 트랜잭션 처리는 서비스에서 담당해야 한다.

```
@Transactional
class SenMoneyService: SendMoneyUsecase{
  //생략
}
```

- - -

테스트
====

- 테스트에는 3가지 종류가 있다: 시스템 테스트, 통합 테스트, 단위 테스트.
  - 비용: 시스템 테스트 > 통합 테스트 > 단위 테스트
  - 테스트 속도: 시스템 테스트 < 통합 테스트 < 단위 테스트. 테스트가 비싸거나 어려우면 커버리지 목표를 낮게 잡아야 한다.
  - 테스트 양: 시스템 테스트 < 통합 테스트 < 단위 테스트

- 단위 테스트는 보통 하나의 클래스에 대해서 테스트한다. 테스트 대상(SUT)이 다른 대상에 의존하면 실제 객체를 사용하거나(classics) mock으로 처리한다(mockist). 이 책에서는 mock을 사용하라는데 본인은 여기에는 동의하지 않는다.
- 영속성 어댑터 테스트는 실제 DB를 대상으로 진행해야 한다.
- 내 개인적인 경험으로는 단위테스트는 도메인 코드에 적용하고 어플리케이션 계층은 통합 테스트를 하는 것이 좋다. 통합 테스트를 할 때에 port에 적절한 실제 객체를 사용하는 것과 mock을 사용하는 것 중에는 고민을 해보는 것이 좋다. 전자의 경우에는 리팩토링할 때 신경을 쓸 필요가 없다는 것과 버그 방지 효과가 높아진다는 장점이 있고, 후자의 경우에는 어댑터를 구현할 필요가 없어진다는 것과 어댑터를 구현하지 않고도 inside-out으로 TDD를 할 수 있다는 장점이 있다. 절충해서 테스트용 어댑터를 구현하는 것도 하나의 방법이다. 포트가 작고 메서드의 수가 적으면 mocking하는 것이 좋을 수도 있다.
- 스프링을 사용한다면 영속성 어댑터 테스트할 때에는 **@DataJpaTest**, 웹 어댑터 테스트 할 때에는 **@WebMvcTest**를 이용하자.

시스템 테스트(E2E)
---------------

- 시스템 테스트는 *요청 -> 웹 어댑터 -> 인바운드 포트 -> 어플리케이션 코어 -> 아웃바운드 포트 -> 영속성 어댑터 -> 어플리케이션 코어 -> 웹 어댑터 -> 응답* 으로 이루어지는 하나의 시나리오를 테스트 한다.

```kt
@SpringBootTest(webEnvironment = webEnvironment.RANDOM_PORT)
class SendMoneySystemTest (private restTemplate: RestTemplate){
  @Test
  void sendMoney(){
    //생략
  }
}
```

- 시스템 테스트할 때에도 서드 파티 시스템이나 외부 서버의 경우에는 Mock을 사용해야 한다.

- - -

경계 간 매핑
==========

- 각 계층사이에서 매핑을 구현해서 사용하는 객체를 격리 시키는 문제는 여러 곳에서 논쟁거리이다
  - 매핑에 찬성하는 입장: 두 계층 사이의 매핑을 하지 않으면 계층 간의 결합이 생긴다
  - 매핑에 반대하는 입장: 보일러플레이트 코드가 너무 많아지고 단순한 CRUD에서 매핑은 과하다.

```
WebController -> Usecase(in port) <= Service -> Repository(out port) <= PersistenceAdapter
     |                  |                |            |                     |
     |                  |                V            |                     |
     -----------------------------> Domain Model <---------------------------
```

매핑하지 않기
---------

- 매핑하지 않는다는 것은 모든 부분에서 도메인 모델을 그대로 사용한다는 것이다. 인어댑터에서 인포트에 제공하는 객체 = 서비스 계층에서 사용하는 객체 = 아웃포트로 제공되는 객체 = 아웃어댑터가 사용하는 객체 = 도메인 모델인 경우.
- 이러한 설계는 각 어댑터의 코드들이 도메인 모델을 더럽힐 수 있다. e.g. JPA를 위한 annotation 또는 웹 어댑터를 위한 Json 설정
- SRP를 위반하긴 하지만 간단한 CRUD usecase에서는 나쁘지 않는 방법일 수 있다.

양방향 매핑
--------

- 양방향 매핑은 각 어댑터에서 각각의 모델을 사용하는 방식이다. e.g. 웹 모델(웹 어댑터에서 사용), 도메인 모델(서비스, 포트에서 사용), 영속성 모델(영속성 어댑터에서 사용)
- 이 방식은 *매핑하지 않기* 전략에서 생기는 문제점인 **더렵혀진 도메인 모델**을 최소화 할 수 있다
- 단점으로는 많은 boilerplate 코드와 도메인 모델이 계층 경계(포트)에서 사용된다는 것이다. 후자의 경우에는 매핑하지 않기 전략보다는 도메일 모델을 더렵혀지는 경우를 최소화할 수 있지만, 그래도 포트를 사용하는 어댑터에 의해서 변경에 취약해질 수도 있다.

```
WebController -> Usecase(in port) <= Service -> Repository(out port) <= PersistenceAdapter
     |                  |                |            |                     |
     V                  |                V            |                     V
  Web Model             -----------> Domain Model <----            Persistence Model
```

완전 매핑
-------

- *완전 매핑*은 *양방향 매핑*에서 조금 더 나아가서 포트에 도메인 모델 대신에 포트만을 위한 DTO를 제공한다.
- 웹어댑터는 웹모델을 웹포트를 위한 객체로 매핑한다. 이 객체는 유효성 검증 로직을 갖는다.
- 서비스에서는 웹포트의 객체를 이용하여 Domain model을 생성하거나 이용한다.
- 이 방식은 많은 객체가 생성되기 때문에 default 패턴으로 사용하지는 않는 것이 좋다. 각 계층 사이에 어떤 동작이 필요할 때 사용하는 것이 좋다. 서비스-어댑터 사이에서 사용하는 것은 의미가 없을 수도 있다. 또는 입력 모델에 대해서만 포트에서 전용 객체를 사용하고 출력은 그대로 도메인 모델로하는 경우도 있다.


```
WebController -> Usecase(in port) <= Service -> Repository(out port) <= PersistenceAdapter
     |                  |                |            |                     |
     V                  V                V            V                     V
  Web Model          Command1        Domain Model   Command2            Persistence Model
```

단방향 매핑
---------

- 이 전략은 모든 계층의 모델들이 같은 인터페이스를 구현하거나 사용한다. 이 인터페이스에는 공통으로 사용하는 데이터들이나 메서드가 들어 있다.
- 도메인 모델은 인터페이스의 정보나 동작 외에도 다른 데이터를 갖거나 서비스 계층에서만 사용하는 도메인 로직을 포함할 수 있다. 이 로직과 데이터는 다른 계층에서 사용할 수 없다.
- 외부에 내부로 들어오는 객체도 실제 도메인 모델로 매핑해서 도메인 모델의 행동에 접근할 수 있다. 이 매핑은 DDD의 팩토리를 통해서 이루어진다. 다른 계층으로부터 객체를 전달 받은 계층은 그 객체를 스스로 자기가 이용할 수 있는 객체로 매핑한다.
- 계층 간의 모델이 비슷한 경우에 효과적이다.

```
WebController -> Usecase(in port) <= Service -> Repository(out port) <= PersistenceAdapter
     |                  |                |            |                      |
     V                  |                V            |                      V
  Web Model             |           Domain Model      |            Persistence Model
     ||                 |               | |           |                      ||
     ||                 -------------|  | |   |--------                      ||
     ||                              V   V    V                              ||
     =============================>   interface  <=============================
```

매핑 선택
------

- 어떠한 매핑 전략을 선택할 것인지는 상황에 따라 다르다. 모든 전략은 각각의 장단점이 있기 때문이다. 만병통치약은 없다.
- 하나의 매핑 전략을 선택하고 끝까지 고정하기 보다는 상황에 따라서 전략을 바꾸는 것이 좋다. 간단한 전략에서부터 필요에 따라 다른 매핑전략으로 변화하는 것이다.
- 예를 들어, 변경 usecase의 경우에는 웹계층과 어플리케이션 계층간의 결합을 제거하기 위해서 웹-어플리케이션 사이에는 완전 매핑 전략을 구현해서 결합도를 낮추고 유스케이스별 유효성 검증도 적용할 수 있다. 그러나 어플리케이션-영속성 계층은 보일러플레이트 코드를 줄이기 위해서 매팡하지 않기 전략을 사용할 수 있다. 반대로 쿼리는 관리해야할 코드를 줄이기 위해서 매핑하지 않기 전략으로 시작해서 양방향 매핑으로 넘어가는 방향으로 넘어가야 한다.

- - -

조립
===
