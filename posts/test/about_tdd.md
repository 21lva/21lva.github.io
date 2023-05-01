---
layout: post
title:  "여러 TDD에 대한 말들"
date:   2022-04-09 02:02:59
author: Inhyuk
category: test
tags: test
cover:  "/assets/instacode.png"
name: about_tdd.md
---


- 여기에는 TDD를 공부하거나 적용하면서 수집한 정보와 책에 대해서 기록하기 위한 공간이다.

Detroit vs London
===============

- TDD에 대한 2가지의 책은 TDD의 큰 2가지 줄기를 형성한다. 하나는 Detroid(a.k.a. Chicago) school의 대표하는 책이라고 할 수 있는 켄트 백의 [테스트 주도 개발](http://www.yes24.com/Product/Goods/12246033)이고, 또 다른 하나는 London school의 프리먼과 프라이스가 저술한 [테스트 주도 개발로 배우는 객체지향 설계와 실천](http://www.yes24.com/Product/Goods/9008455)이다.
- 양측 다 방식의 차이는 있지만 TDD를 선호한다.
- 이 둘은 크게 3가지에서 차이를 보인다.
  1. **단위 테스트**를 바라보는 관점: 단위 테스트란 하나의 단위를 빠른 시간안에 다른 테스트와 격리된 상태에서 테스트하는 것이다. 여기서 중요한 것은 격리가 되는 단위이다. 두 스쿨은 단위에서 다른 생각을 갖고 있다.
  2. test를 작성하는 방법: Application을 만들기 위해서는 desgin하고 UML을 만드는 과정을 거쳐야한다. TDD는 이런 과정을 위한 새로운 방식이 될 수 있다. 즉, 테스트를 통해서 디자인을 변화시킬 수 있다는 말이다. 두 스쿨은 TDD를 통한 디자인에 대한 방식의 차이가 존재한다.
  3. Mock, stub과 같은 test double에 대한 호불호 등에서 여러 차이가 존재한다.


||작은 코드|격리|test double 대상|
|-|-|-|-|
|런던|클래스,메서드|협력자|불변 의존성외 모든 의존성|
|고전|동작|다른 테스트|공유 의존성|


Detroit School
---------------

- 단위 테스트의 단위는 **모듈**을 의미한다. 모듈은 하나의 클래스, 함수, aggregate등 다양하다. 즉, 하나의 동작을 테스트한다고 보면 된다.
- application을 설계하기 위해서 core logic에 집중한다. 그 후에 bottom-up 방식으로 API까지 설계한다. 설계하다가 협력자가 필요한 경웨 협력자를 구현한다.
- test double의 사용을 최소화한다. 내부로부터의 설계는 어떤 *내부* 협력자가 필요할 때 이미 그 내부 협력자의 구현이 완성되었다는 말이다. 즉, 그 내부 협력자에 대한 mock이 필요 없다는 의미이다. test double을 과도하게 사용한다면 class끼리의 상호작용을 제대로 테스트할 수 없게 되고 그러면 동작을 테스트한다는 의미에서 테스트의 효용성에 의문이 생길 수 밖에 없다고 한다.
- mock을 전혀 사용하지 않는 것은 아니다. 외부에서 다른 사람과 공동으로 사용하는 의존성의 경우에는 mock으로 처리한다.
- E2E 테스트, 통합테스트를 최대한 적게 하라고 한다.
- 테스트가 프로그램의 동작의 변경에는 민감해야 하지만, 구조의 변경에는 민감하지 않아야 한다고 한다.
- 상호작용보다는 동작의 결과(return값 혹은 state)를 검증한다
- 단점
  - bottom-up 방식은 실제 사용자가 원하는 API와 다른 over-design의 위험이 있다. YAGNI에 위배
  - test가 실패하는 경우에 코드의 어느 부분에서 오류가 발생한 것인지 알기 어렵다.

London School
---------------

- 단위테스트에서 단위는 하나의 클래스이다. 그러므로 이 클래스를 테스트하기 위한 다른 클래스들은 모두 mock으로 처리해야 한다.
- 가장 외각의 API를 설계하고 그로부터 top-down형식으로 core logic까지 설계한다. 설계하다가 협력자가 필요한 경우 구현은 하지 않고 Mock을 이용하여 처리한다.
- *Mockist*라고 불릴 정도로 Mock을 적극적으로 사용한다. 현재 클래스를 제외한 다른 협력자는 mock으로 처리한다.
- E2E테스트와 통합테스트로 mock처리로 알 수 없었던 클래스끼리의 협력을 테스트하라고 적극 권장한다.
- 단위 테스트들은 서로 독립적이어야 하기 때문에 다른 협력자들도 mock하지 않고 테스트하는 방식을 통해서 원하지 않은 테스트까지도 깨지는 경우를 싫어한다.
- **interaction-based test**를 선호한다: 오브젝트끼리의 상호작용을 메시지를 제대로 보냈는지를 통해서 테스트 한다.
- layer단위로 내려가기 때문에 각 layer를 testable한 구조로 만들 수 있다.
- API 단위로 설계하기 때문에, API 변경시에 Inside-out 방법보다 더 적은 코드 변경이 이루어진다.
- API 설계는 명확하지만 도메인 설계가 명확하지 않을 때 사용하기에 좋다
- 이 방식은 third-party 라이브러리를 이용한 코드를 테스트할 때 유용하다. 또한, 신뢰되지 않는 레거시 코드를 이용하는 경우에도 유용하게 사용할 수 있는 방법이다. [관련 내용](https://itnext.io/outside-in-or-inside-out-part-2-refactoring-existing-projects-dcf7f2ef4d2a)
- 단점
  - 너무 과도한 mock을 사용하면서 **white box testing**을 만들게 되는데 이는 테스트들이 구현에 너무 과하게 결합되면서 **fragile test**를 양산할 수 있다


TDD에 대한 얘기들
-------------

#### 의존성이 너무 많아서 TDD 하기 쉽지 않다

- 의존성으로 인해 테스트하기 어려운 코드가 생성된다는 말은 나쁜 아키텍처의 신호이다.

#### Logic이 복잡해서 TDD 하기 어렵다

- 로직이 복잡할 때는 로직을 이해하도록 도와주는 테스트를 작성하고 완성이 된 후에 테스트를 삭제해도 무방하다.

- - -

Mock을 사용하지 않은 integration test
=============

- [원글](https://phauer.com/2019/focus-integration-tests-mock-based-tests/)을 정리하였다.

- Mock을 사용하는 테스트는 상술한 이유에서 refactoring하기 힘들고 integration test의 역할을 제대로 수행하지 않는다. Mock 객체 대신에 실제 객체를 사용하는 방법은 모든 layer를 거치는 테스트를 진행할 수 있다. in-memory DB 대신 실제 DB를 사용하는 방법을 통해서 더 나은 버그 방지 기능을 얻을 수 있다.

Example
------

- **GET /products**를 통해서 product 정보를 얻는다고 해보자. 이 때 일어나는 일은  
  1. request 받음
  2. DB에서 product 객체 받아옴
  3. Tax 서버로부터 정보를 전달 받음
  4. Price 계산 로직 수행
  5. response 전달
- 구조는 다음과 같다
  1. ProductController: 요청을 전달 받는 객체. ProductDAO, TaxServiceClient, PriceCalculator를 사용
  2. ProductDAO: DB에 접근하는 객체. In-Memory DB를 사용한다
  3. TaxServiceClient: Tax 서버로 요청을 하는 객체
  4. PriceCalculator: Price를 계산하는 로직을 포함한 객체

- Mockist style test는 각 객체를 테스트 하기 위해서 객체가 의존하는 모든 의존성을 **Mock**으로 만든다. e.g. ProductController를 테스트하기 위해서는 ProductDAO, TaxServiceClient, PriceCalculator를 Mock으로 만든다. 이 방법은 결국 다음과 같은 문제를 야기한다
  1. 테스트의 범위가 한정된다. ProductController Test를 통과한다고 해서 실제 서비스에서 ProductController로 들어온 요청이 제대로 처리된다는 보장을 할 수 없다.
  2. 리팩토링시 고통이 동반된다. ProductController에서 만약에 TaxServiceClient가 변경된다면 모든 Mock을 변경해줘야하고 이는 깨지기 쉬운 테스트라는 의미이고 고통스러운 리팩토링을 야기한다.
  3. 실제 DB를 사용하지 않기 때문에 실제 DB와의 차이로 인한 문제를 테스트하지 못한다.
  4. Test class가 너무 많아진다.

해결 방법
-------

- 위의 문제들을 해결하는 방법은 하나의 integration Test 클래스를 통해서 내부의 실제 객체들을 테스트 하는 것이다. 이 때 DB는 실제 DB를 사용한다.
- 실제 DB를 사용하는 것과 달리 관리되지 않는 외부 의존성인 Tax Server는 그대로 Mock Server를 사용해야 한다. 만약, Tax Server 쪽에서 테스트를 위한 서버를 제공한다면 사용해도 무방하지만 그래도 되도록이면  Mock Server를 사용하자.
- 실제 DB와 같은 DB를 docker container레 띄워서 사용하자. [Test Containers](https://www.testcontainers.org/)에서 java api로 docker를 이용한 테스트를 쉽게 할 수 있다.
- 이 방법은 다음과 같은 장점이 있다.
  1. 실제 객체들과 DB를 사용하기 때문에 실제와 가장 비슷한 테스트를 하게 되고, 버그를 최대한 방지할 수 있다.
  2. 내부 구현에 의존하지 않기 때문에 리팩토링에대해 안전하다.
  3. Test class가 너무 많아지는 것을 방지한다. 통합테스트는 단위 테스트와 달리 가장 많은 의존성을 테스트하는 길지만 적은 테스트 구조를 유지해야한다
  4. 실제 DB를 사용함에도 불구하고 Java code를 이용하여 쉽게 테스트 환경을 구축할 수 있다.

해결해야할 문제들
------------

#### 테스트 속도

- integration 테스트의 가장 큰 문제는 결국 테스트의 속도에 있다.
- Mock을 사용하나 실제 객체를 사용하나 테스트의 속도에는 큰 차이는 없다.
- 속도 측면에서 문제가 될 수 있는 부분은 2가지이다
  1. Spring과 같은 무거운 DI framework: 실제로 가장 크게 신경쓰이는 부분. 저자는 DI를 최대한 자제하고, lazy initialization등을 이용하여 framwork 사용을 최소화 하는 방법을 이용한다고 한다.
  2. DB를 위한 Container: 피할 수 없다. 전체 테스트 당 한 번의 Container 생성을 하도록 하자.

#### Input data와 output data 생성

- input data와 그에 맞는 output data를 만드는 것은 귀찮다.
- **createProductEntity**와 같은 method를 이용하여 기본 값을 설정하고 객체를 생성하도록 하자.

#### Corner case 테스트를 하기 어려움

- 만약 **ProductDAO**의 *findProducts()* 메서드를 테스트하고자 한다면, integration test를 이용하는 것은 *overkill* 이 될 수 있다.
- 그렇지만, corner case를 위한 테스트도 작성해야 한다. 기존의 integration test와 위의 객체 생성 *helper* 메서드등을 이용하면 테스트 작성이 크게 어렵지 않다.
- 또한, 실제로 위의 생각은 명확히 얘기해서 *findProducts()* 를 사용하는 상황(behaviour)들을 테스트 하고 싶은 것이다. e.g. 특정한 경우에 *findProducts()* 를 호출하면 안되는 사용자가 호출하는 경우 등등

#### integration test에서 특정 상황을 테스트하지 못하는 경우

- 어떤 상황을 integration test를 통해서 테스트 하지 못하면 코드 구조가 잘못 되었거나 일어나지 않을 상황을 테스트하고자하는 것일 수도 있다.

- - -

Test-induced design damage
===================

- [Test-induced damage By David Heinemeier](https://dhh.dk/2014/test-induced-design-damage.html)에 대해 정리한다.
- [관련 토론](https://gist.github.com/dhh/4849a20d2ba89b34b201)

원글
-------

- TDD를 주장하는 사람들은 "고립된 테스트(단위테스트??)를 하지 못하는 코드는 잘못 설계된 것이다."라고 주장한다. 여기서 고립은 DB나 file IO와 같은 느린 외부 의존성에 의존하지 않는다는 것을 의미한다.
- 이 주장은 특정 상황에서는 맞는 얘기이지만, 항상 그렇다고는 할 수 없다. 위의 주장을 따르면 빠르게 단위 테스트할 수 있는 코드를 만들 수는 있지만, 필요 없는 간접참조와 오버헤드를 야기하는 **test-induced design damage**를 초래할 수도 있다.
- **TDD를 맹신하는** 사람들은 이러한 경우를 무시하고 있다. 그러나 많은 다른 사람들은 "TDD is dead"라고 말하고 있다.

#### TDD와 hexagonal design으로 부터 생길 수 있는 문제들

- [Rail을 통한 hexagonal architecture를 설계하는 방법 영상](https://www.youtube.com/watch?v=tg5RFeSfBM4)에는 실제 DB와 외부 context에 대한 의존 없이도 쉽고 빠르게 테스트하는 방법이 나온다.
- 여기서는 controller(application service)가 직접적으로 DB에 접근하는 객체를 사용하지 않고, *Repository(out-bounding port)* 를 통해서 간접적으로 사용한다고 한다. 이 방법은 **빠른 테스트**, mock으로 단위 테스트하기 쉬운 코드로 인한 design적인 문제를 갖고 있다. 이 2 가지 목표의 기저에 깔린 목적은 application과 Rails 사이에 경계를 만드려는 목적과 같다.
- **hexagonal desing pattern** 그 자체가 문제인 것은 아니다. 이 패턴은 웹앱의 도메인 로직을 외부로부터 격리하는 아주 좋은 방법이 될 수 있다. 그러나 이 패턴은 **port-adaptor pattern**과 같이 테스ㅡ 목적으로 잘못 이용되고 있다. 이는 **Test-driven pattern application**이다.
- 실제로 Repository와 같이 직접적인 접근을 막는 패턴등은 오직 빠른 테스트를 위해서 만들어진 패턴이고 이 방법은 실제 어플리케이션을의 디자인을 명확하게 표현하지 않는다.

#### "Unit test are best"라는 생각을 버리자

- 내(원글 저자)가 생각하기로는 이러한 문제가 발생한 이뉴는 **Controller**(application service)를 단위테스트하려 하기 때문이다. Controller의 목적은 request와 response를 session의 context 내에서 **통합**하는 것이다.
- **Controller**는 **unit test**의 대상이 아니라 **integration test**이다. 또한, **View**도 비슷한 이유로 E2E테스트의 대상이다
- DB를 이용하는 테스트에 대한 공포는 옛날 이야기이다. 이러한 종류의 **decoupling**은 더 이상 의미가 없다. 이제는 실제 DB를 이용하는 테스트도 충분히 빨라졌다.

- 이 시점에서 **BDD**를 주장하는 사람들은 단위 테스트는 필요 없다고 말할 수도 있다. 그러나 "test 먼저"라는 생각 아래에서는 이러한 주장도 크게 도움이 안되고 많은 mocking과 인위적인 경계 설정으로 받는 고통을 경감 시켜주지 않는다.
- 내가 생각하기로는 Rails/MVC testing의 핵심은 아래와 같다
  1. 내부 로직을 위해서 model을 테스트 하고, 너무 DB에서 분리하려는 하지 마라.
  2. Controller는 integration test를 하자. 권한 체크와 같은 session logic을 command object로 추출해서 mock boundary를 생성하고 테스트를 빠르게 하는 것을 하지말도록 하자. 이것은 우리가 하고자하는 anemic controller가 아니다.

- 가장 중요한 것은 어느 부분에 집중할 것인지이다. 만약 너가 간단한 모델 + 복잡한 UI로 이루어지는 application을 만들고자 한다면, model test보다 system test에 집중해야 한다. 또한, 너의 controller가 딱히 하는 일이 없다면 controller를 test-drive로 구성할 필요 없이 system test로 대체할 수 있다.
- 즉, 너의 모든 디자인을 test-driven으로 구성하는 것은 지양하고 디자인을 바탕으로 test를 구성하는 방법을 생각해야한다. 시스템의 design integrity는 특정 layer를 테스트하는 것보다 중요하다.

- - -

TDD IS DEAD
==========

- - -

David Heinemeier와 Kent Beck의 대화 정리
================================

- [둘 간의 대화 정리 by Martin Fowler](https://martinfowler.com/articles/is-tdd-dead/)
에서는 두 사람간의 TDD에 대한 대화를 정리해본다.

어디서 이 대화가 시작되었는가
-----------------------

- 이 대화는 [Daivd의 keynote](https://dhh.dk/2014/tdd-is-dead-long-live-testing.html)에서 시작되었다. 이 글에서 그는 TDD와 Unit test에 대한 불만을 표출하였다. Martin은 이 글에 대해 오타를 수정하도록 하면서 David와 여러 이야기를 나누었다. David, Martin, Kent는 대화를 통해서 서로의 경험과 관점에 대해 알고 싶었다.

TDD와 자신감
---------

- David는 TDD와 unit test에 대해서 3 개의 생각을 개진했다.
  1. TDD의 정의에 대한 혼동
  2. architecture를 구성하기 위해서 mock을 사용하면서 야기되는 test-induced damage
  3. Red-Green-Refactor가 왜 자신에게는 맞지 않았는지
- Martin은 TDD를 이해하기 위해서는 TDD의 역사에 대해 잘 알아야한다고 생각하였고, Kent가 TDD가 어떤 과정을 거쳐서 만들어지게 되었는지를 설명하였다. 처음에는 TDD를 사용하지 않았고, code에 자신감을 얻기 위해서 선택한 하나의 방법이 TDD라는 것이다.
- David는 Test의 중요성에 대해서는 동의하지만, TDD가 가고자하는 길은 동의하지 않았다. 그는 TDD가 self-testing code에서 얻는 자신감과 TDD를 결합하는 것은 좋아하지 않았다.
- Kent는 최근 Facebook에서 진행한 해커톤에서 TDD를 적용할 수 있는 경우와 그렇지 않은 경우에 대해서 이야기하였다. 전장의 경우에는 즐거웠지만 후자의 경우에는 조금 까다로웠다. non-TDD의 경우에도 그는 regression test와 짧은 feedback loop을 사용하였으며 둘을 섞는 것이 문제가 되지 않았다고 한다. TDD는 그가 학교에서 예시를 통해서 수학을 배운 것처럼 생각하는 방법을 제시한다고 하였다.
- David는 TDD가 적용되는 경우도 있었지만 대부분의 경우에는 그렇지 못하다고 하였다. 그는 TDD를 위해서 어떤 희생을 하고자하냐고 물어봤다. 대부분의 사람은 과도한 mocking 같은 아주 안좋은 trade-off를 하고 있다고 하였다.
- Kent는 위의 주장은 trade-off와 관련되어 있다고 생각하였다. 그는 compiler의 예시를 통해서 중간단계에서 테스트 가능한 코드가 가치가 있다고 주장하였다. 그러나 Mock과 관련되어서는 Kent는 Mock을 거의 사용하고 있지 않으며 그것을 사용하는 것이 리팩토링을 어렵게한다고 하였다.
- Martin이 생각하기로는 David의 TDD에 대한 비판은 주로 과도한 mocking 사용에 대한 것이며 self-testing code와 TDD는 다르다고 한다. TDD는 단지 self-testing code를 얻는 한 방법론일 뿐이다.

Test-induced design damage
---------------------

- David는 위의 글에서처럼 **hexagonal rails**와 같은 TDD 접근방법은 과도한 간접접근을 통한 test-induced design damage를 야기할 수 있다고 하였다. 그러나 Kent는 이것은 TDD와 관련 없고 design decision과 관련된 문제라고 하였다.
- Martin은 3개의 질문을 David에게 하였다: 1) TDD가 design damage를 야기하는가? 2) 그것은 정말로 design damage인가? 3) 어떻게 design이 damage되었는지를 알 수 있는가?
- David는 [예시](https://gist.github.com/dhh/4849a20d2ba89b34b201)에서 Mock을 사용하는 TDD를 통해서 격리된 테스트를 위한 과도한 간접접근과 복잡함에 대한 설명을 하였다. 이 예시에서 각 layer는 부린되어 있었다. 예를 들어, controller는 실제 model과 db, request/response와 상호작용 없이 테스트 가능하였다.
- Kent는 TDD를 통해서 test-induced damage가 발생하였다는 것은 자동차를 잘못된 곳으로 운전하고서는 자동차를 탓하는 것과 같다고 하였다. 그는 David가 보여준 예시는 TDD로 인해서 발생한 문제가 아니라고 설명하였다. 위의 예시에서 사용하는 간접접근은 특정 상황에서 좋은 방법이고 그러한 상황에 대한 분별을 해야한다고 하였다.
- David는 Kent의 의견에 동의하지 않으면서 TDD를 하게 되면 결국에는 이와 같은 design을 하게 된다고 하였다.
- Kent는 TDD는 design에 대해서 점진적인 압력을 주고 사람마다 test를 통해서 처리하는 크기에 대한 선호도가 사람마다 다르다고 하였다.
- David는 code의 사이즈와 그것을 얼마나 쉽게 바꿀 수 있는지는 직접적인 상관관계가 있다고 하였다. 직접적인 참조로 만든 코드보다 간접 참조로 만든 코드가 더 길어지게 되고 결국에는 모든 layer에서의 간접참조는 큰 비용으로 다가올 수 있다고 하였다. 또한, TDD의 red->green->refactor 사이클은 사람들을 안좋은 선택을 이끈다고 하였다.
- Martin은 David의 생각에 동의하지 않았다. 그가 생각하기로는 David가 말한 문제는 TDD로부터 발생한 문제가 아니라 hexagonal architecture에서 다른 환경에서 격리되고자하는 본질때문에 발생한 문제라는 것이다.
- David는 사람들이 isolation을 하고자하는 이유가 TDD 때문이라고 하였다.
- Kent는 David가 말한 코드 길이 문제는 cohesion과 관련된 것이라고 하였다. David는 여기에 동의하였지만, cohesion과 coupling은 자주 반대되는 경우라고 하였다. 결합도가 높으면 좋은 응집도를 위한 비용지불일 수도 있다고 하였다.
- Kent는 외부 의존성을 제거하는 다른 방법들이 존재한다고 하였다. 그는 test하기 어렵다는 것은 design에 대한 통찰력이 필요하다는 지표라고 하였다.
- David는 test가 좋은 디자인을 만들어 줄 수 도 있다고는 생각하지만, 그의 경험상 반대로 좋은 design이지만 테스트하기 힘든 경우도 있다고 하였다.
- Kent는 David에게 자신감이 부족하다고 비난(?)하였다. 지금은 통찰력이 부족할 수 있지만 노력하다보면 추후에는 알 수 있을 것이라고 하였다.
- David는 오히려 Kent를 **faith-based TDD**라고 일축하였다. 그는 TDD를 사용하고 싶었지만 이상한 loop에 빠지게 된다고 하였다.
- Kent는 David가 TDD가 아니라 software design의 전반적인 얘기를 하고 있다고 말했다.

Feedback & QA
---------------

- Kent는 TDD와 관련된 결정들은 trade-off라고 하였다. 이상적인 세계에서는 프로그래밍적인 결정에 대한 즉각적이고 문제 없는 피드백이 오지만, 현실은 그렇지 않다고 하였다. 그렇기 때문에 얼마나 이상에서 떨어진 현실을 받아들일 것인지를 결정해야한다. 이를 위해서 다음과 같은 조건에 대해 고민을 해야한다.아래의 네 가지 조건은 trade-off를 하기 위해서 고려해야할 것들이다.
  1. Frequency: 얼마나 자주 피드백을 받을 것인가?
  2. Fidelity: 얼마나 정확한 피드백을 받을 것인가?
  3. Overhead: 어떤 비용을 치룰 것인가?
  4. Lifespan: 이 소프트웨어를 얼마나 오래 사용할 것인가?

- David는 TDD의 성공이 QA에 대한 무시를 야기했다고 말한다. Kent는 QA와의 오래된 관계가 제대로 동작하지 않는다고 말했다. 그는 "페이스북의 어떤 것도 남의 문제가 아니다."라는 말을 예시로 들었다. 페이스북은 최근까지 QA가 없었다고 말했다. 그는 좋은 QA > no QA > 제대로 동작하지 않는 QA라고 말했다. 마틴은 제대로 동작하지 않는 적대적인 관계를 제거하는 것보다도 수동으로 동작하는 QA를 제거하는 것이 중요하다고 생각한다. 이렇게 하면 startup에서 QA없이 작동할 수 있게 된다고 하였다. David는 초기에 QA를 하지 않으면서 얻는 속도적인 이득에는 동의하지만 exploratory testing에 대해서 무시해서는 안된다고 말하면서 QA 없이 고품질의 소프트웨어를 만들 수 있다고 생각하면 잘못된 생각을 하고 있다고 말했다. 실제 프로덕션 단계에서 사용자는 어떠한 이상한 일을 할지 모른다는 것이다. 특히, 개발자가 고객 서비스와 아무 관련 없을 때 문제가 최악이 될 수 있다고 하였다. 많은 프로그래머는 on-call을 싫어하는데 이는 또다른 피드백 루프이다. Kent는 on-call이 개발자들에게 작성하지 않은 테스트를 새로 작성해야한다는 사실을 알려준다고 하였다. 많은 개발자들이 이것을 싫어하지만 이 것은 벗어날 수 없으며 실수하지 않았다고 확신하는 순간이 실수하는 순간이고 성장을 멈추는 생각이라고 하였다.
- trade-off를 이해하기 위해서는 cost에 대해서 알아야 한다. 비용에 대해서 생각하지 않는 것은 test-induced damage가 있다는 것을 이해하지 못하는 이유이다. 안정성 비용을 고려해야한다. 99%에서 99.99%로 가는 비용이 0%에서 99% 가는 비용보다 더 비쌀 수도 있다.

Costs of Testing
-----------------

- David는 trade-off에 대해 알려면 drawback에 대해서 잘 알아야 한다고 하였다. 그는 TDD가 그러한 생각을 하지 못하게 한다고 하였다. 특히 그는 테스트 없이 코드를 작성하지 않는다는 행위가 **over-testing**으로 이어지고 이 over-testing이 결국에는 behavior 변화에 필요한 코드 수정이 늘어나는 현상을 발생시킨다고 하였다. Kent는 이제까지 test를 억지로 작성하지 말고 자신감이 생길 때까지 작성하라고 하였는데 David는 Martin과 Kent에게 노테스트노코드 원칙을 지키고 있는지 물었다.
- Kent는 상황에 따라 다르다고 하였다. JUnit을 사용하면서 테스트 우선에 엄격하더라도 over-testing을 하는 경우는 없었다고 하였다. **Delta coverage**(이 테스트가 유일하게 제공하는 coverage가 무엇인가에 대한 지표)가 0인 테스트는 다른 개발자들과의 의사소통을 위한 것이 아니면 의미가 없으며 제거되어야 한다고 말하였다. 그는 종종 테스트 작성 -> 구현 -> 리팩토링 -> 테스트 제거를 한다고 하였다. 많은 사람들이 테스트를 제거하는 것에 대해서 두려워 하는데 그것은 꼭 필요한 과정이다. 하나의 기능이 어려 곳에서 테스트된다면 결합도가 높아지고 비용이 증가한다는 것이다.
- Martin은 over-tested code가 존재한다고 하였다. 그는 그러한 경우가 너무 과하지 않다면 크게 문제되지 않는다고 하였다. **테스트 하나에 한 줄 관점**에서 그는 "이 코드를 바꾸면 테스트가 실패할 까?"를 자문한다고 하였다. 주석처리나, 문장을 제거해서 테스트가 실패하는지를 확인한다.
- Kent는 test당 production code 비율은 의미없는 지표라고 하였다. David는 주석처리를 발견한다는 것은 100%의 테스트 coverage를 의미한다고 하였다.
- Martin은 코드를 자신있게 리팩토링 할 수 없으면 테스트가 충분하지 않다고 말했다. 테스트가 너무 많다는 것은 변화가 있을때마다 프로덕션 코드보다 테스트를 더 많이 수정해야 할 때 알 수 있다고 하였다.
- David는 이제까지 사람들이 코드보다 문서를 중요하게 생각했던 것처럼 테스트를 프로덕션 코드보다 중요하게 생각할까봐 걱정된다고 하였다. TDD 사이클에서 리팩토링 부분을 덜 중요하다고 생각하게 된다는 것이다. Kent는 테스트는 남겨놓고 프로덕션 코드를 지우고 다시 작성했던 경험을 얘기하면서 "코드를 남겨놓고 테스트를 버리는 경우"와 "테스트는 남겨놓고 코드를 버리는 경우" 중 어느 것을 선호하는지 질문했다.
- Martin은 테스트를 읽는 것이 코드가 하는 행동을 유추하는 데에 크게 도움이 된다고 하였다.


- - -

Test double
===========

- Test double에는 다음과 같은 종류가 있다
- Mock
  1. Mock: command 요청 테스트에서 행위를 검증할 때 사용한다. 어떤 command 요청이 들어오면 적절한 처리 후의 결과를 전달한다.   
  2. Spy: 호출된 내용에 대한 정보를 기록한다. Stub의 역할을 포함한다
- Stub
  1. Dummy: 아무것도 하지 않는 객체. 단순히 객체가 필요할 때 사용한다
  2. Fake: 실제 객체가 아닌 단순 동작만을 하는 객체. stub과 같은 일을 하지만 차이점은 실제로 동작하지 않는 의존성에 대해서 fake를 만든다.
  3. Stub: query 요청에서 필요한 데이터를 전달하는 객체. 상태 기반 테스트에서 사용된다.

- fake vs stub

```
A Fake is closer to a real-world implementation than a stub. Stubs contain basically hard-coded responses to an expected request; they are commonly used in unit tests, but they are incapable of handling input other than what was pre-programmed.

Fakes have a more real implementation, like some kind of state that may be kept for example. They can be useful for system tests as well as for unit testing purposes, but they aren't intended for production use because of some limitation or quality requirement.
```

- - -

Mocks are not stubs
==================

- [원글](https://martinfowler.com/articles/mocksArentStubs.html)
- 많은 곳에서 실제 객체를 모방하기 위해서 **Mock**을 사용한다. 그러나 많은 사람들은 Mock이 단순히 test object 중 하나이며 다른 object들로 오용하고 있다. 특히, **Stub**과 **Mock**사이의 혼동은 자주 있다.
- 둘 사이의 가장 큰 차이는 2가지이다: 상태 검증-행동 검증, TDD의 두 style에서 발생하는 관점

Regular tests
--------------

- order 객체를 받아서 warehouse object로 채워 넣는 예시를 보자. order 객체는 product 하나와 quantity 하나로 이뤄져있다. warehouse는 여러 product들을 가지고 있다. 2 가지의 시나리오 가능하다 1) order만큼의 product가 warehouse에 있으면 warehouse의 양을 줄이고 order의 양을 늘린다. 2) 불가능하다면 아무 일도 일어나지 않는다.
- 아래의 test는 전형적인 AAA(arrange, act, assert)구조로 되어 있다.

```
void setUp() throws Exception {
  warehouse.add(TALISKER, 50);
  warehouse.add(HIGHLAND_PARK, 25);
}

public void testOrderIsFilledIfEnoughInWarehouse() {
  Order order = new Order(TALISKER, 50);
  order.fill(warehouse);
  assertTrue(order.isFilled());
  assertEquals(0, warehouse.getInventory(TALISKER));
}

public void testOrderDoesNotRemoveIfNotEnough() {
  Order order = new Order(TALISKER, 51);
  order.fill(warehouse);
  assertFalse(order.isFilled());
  assertEquals(50, warehouse.getInventory(TALISKER));
}
```

- Arrange(setup 포함)단계에서 Order객체(SUT)와 Warehouse객체(Collaborator)를 생성한다. Warehouse 객체는 Order 객체가 필요로하는 책임을 제공하고 Warehouse를 검증하는 데에 사용된다. 후자와 같은 테스트를 SUT의 행동 이후에 collaborator의 메서드가 작동하였는지를 확인한다는 의미에서 **상태 검증**이라고 한다.

Tests with mock objects
-------------

- 이번에는 같은 동작을 테스트하지만 **Mock**을 사용하는 예시를 살펴보자.

```
public void testFillingRemovesInventoryIfInStock() {
  //setup - data
  Order order = new Order(TALISKER, 50);
  Mock warehouseMock = new Mock(Warehouse.class);

  //setup - expectations
  warehouseMock.expects(once()).method("hasInventory")
    .with(eq(TALISKER),eq(50))
    .will(returnValue(true));
  warehouseMock.expects(once()).method("remove")
    .with(eq(TALISKER), eq(50))
    .after("hasInventory");

  //exercise
  order.fill((Warehouse) warehouseMock.proxy());

  //verify
  warehouseMock.verify();
  assertTrue(order.isFilled());
}

public void testFillingDoesNotRemoveIfNotEnoughInStock() {
  Order order = new Order(TALISKER, 51);    
  Mock warehouse = mock(Warehouse.class);

  warehouse.expects(once()).method("hasInventory")
    .withAnyArguments()
    .will(returnValue(false));

  order.fill((Warehouse) warehouse.proxy());

  assertFalse(order.isFilled());
}
```

- 위의 코드를 보면 기존과 달리 setup 단계가 2개로 나누어져있음을 알 수 있다: 기존의 setup + **expectations**. 기존에는 실제 객체를 사용하였지만 여기서는 Mock 객체를 사용혀아ㅕㅅ다.
- SUT의 동작을 수행하게 한 후에 verification을 해서 Mock이 우리가 원하는 행동을 하였는지를 확인한다. 기존 테스트에서는 테스트 이후에 warehouse의 상태를 검증하였지만, 이번에는 warehouse가 동작을 수행했는지를 검증하는 **behaviour verification**을 수행한다.

The difference between Mocks and Stubs
----------------------------

- 사람들은 Mock, Stub, Fake 등을 혼용한다. 여기서는 Gerrad Meszaror의 책에 기반을 해서 **Test Double**들을 정의한다.

1. Dummy: 제공되지만 절대 사용되지 않는 객체. 보통 parameter을 채우기 위해서 사용된다.
2. Fake: production 단계에서 사용되지 않고 단순히 어떤 객체를 대신하도록 사용되는 객체
3. Stub: 어떤 요청에 대한 응답을 제공하는 객체.
4. Spy: Stub + 자신이 제공한 응답과 그 요청을 기록하는 기능을 제공하는 객체
5. Mock: 어떤 요청에 대한 기대를 미리 정해놓는 객체. 유일하게 behavior verification을 할 수 있는 기능을 제공한다.

- Mock이 아닌 다른 객체들은 state verification에 사용되고 Mock, fake, stub, spy 등은 사용자에게 적절한 대답을 제공하지만 오직 Mock만이 어떤 동작이 기대되는지를 정할 수 있다.
- 많은 사람들은 test double을 단순히 우리가 알지 못하는 어떤 객체를 대체하여 적절한 응답을 제공하도록 사용한다. e.g. email server에 요청을 보내는 객체를 테스트할 때 email server 대신 test double을 사용하지 않으면 실제 email server에 요청을 보내는 불상사가 발생할 수 있다.
- 아래의 예시는 위에서 설명한 Stub와 Mock을 사용하여 state verification과 behavior verification을 수행하는 테스트 2개이다

```
//Stub
public void testOrderSendsMailIfUnfilled() {
  Order order = new Order(TALISKER, 51);
  MailServiceStub mailer = new MailServiceStub();
  order.setMailer(mailer);
  order.fill(warehouse);
  assertEquals(1, mailer.numberSent()); //state verification
}

//Mock
public void testOrderSendsMailIfUnfilled() {
  Order order = new Order(TALISKER, 51);
  Mock warehouse = mock(Warehouse.class);
  Mock mailer = mock(MailService.class);
  order.setMailer((MailService) mailer.proxy());

  mailer.expects(once()).method("send"); //behavior verification을 위한 세팅
  warehouse.expects(once()).method("hasInventory")
    .withAnyArguments()
    .will(returnValue(false));

  order.fill((Warehouse) warehouse.proxy());
}
```

Classicial and Mockist testing
-----------------------

- 위에서는 test double들을 바라보는 첫 번째 관점인 state verification vs behaviour verification에 대해서 설명하였다. 여기서는 그 다음 관점인 TDD에서의 test double대해서 알아보겠다.
- Classical TDD는 가능한 실제 객체를 사용하며 test double을 사용하는 경우는 위에서 예시로 든 email server와 같이 불가피한 경우에만 한정된다.
- Mockist TDD는 어떤 객체가 의존하는 모든 의존성에 대해서 test double을 사용한다.
- BDD는 Mockist style에서 파생된 방식이다. 원래 BDD는 TDD가 어떻게 하면 design technique에서 동작할 수 있는지를 보여주면서 TDD를 배우기 쉽게 하기 위해 만들어졌다. 이것은 test를 behaviour라고 변경되면서 TDD가 어떤 객체의 책임을 고려하도록 하였다. BDD는 Mockist style을 기반으로 하지만 naming style과 기술적인 분석을 통합하려는 욕구를 바탕으로 조금 더 확장된다.

Choosing between the differences
----------------------

- state vs behavior, classicist vs mockist 중 어떤 것을 선택해야 할까?
- 먼저 state vs behaviour이다. 이 때 가장 먼저 고려해야할 것은 문맥(context)이다. 어떤 collaborator가 간단하게 사용할 수 있는 객체인 경우에 classicist는 실제 대상을 사용할 것이고, mockist는 test double을 사용할 것이다. 만약에 collaborator가 사용하기 까다로은 경우라면 mockist는 여전히 test double을 사용할 것이고 classicist는 2가지 선택지(실제 대상을 사용하거나, test double을 사용하거나) 중에 고민을 할 것이다. 이때 classicist들은 각 상황에 맞게 가장 쉬운 선택을 하게 될 것이다. 결국 state냐 behavior냐에 대한 고민은 자신이 classicist이냐 mockist 방식을 따르냐에 따라 달라지는 문제이다.


#### Driving TDD

- Mock은 XP community를 통해서 만들어졌는데 이 커뮤니티가 가장 중요하게 생각한 부분은 TDD이다. 특히, TDD를 통해서 어떻게 시스템을 구성할 것인가에 대한 고민이었다.
- Mockist들은 **need-drivient development**를 주장한다. 이 방식은 out->in 방식으로 user story(usecase)의 test를 작성하는 것 부터 시작한다. 이후에 협력자에 대한 interface를 고민하고 여기에 mock을 적용한다. 첫 번째 테스트를 위해 작성한 mock은 다음 객체를 만드는 데에 필요한 조건들과 시작점을 제공한다.
- Classicist는 어떤 특정한 가이드를 제공하지는 않는다. Mockist처럼 outside-in 방식으로 개발하고자 한다면 Mock 대신에 stub을 사용한다는 것이 큰 차이이다. 어떤 collaborator가 필요한 경우에 단순히 하드코딩을 한다. 추후에 실제 객체가 구성된다면 하드코딩된 부분을 다른 적절한 객체로 대체한다. 물론 다른 방법으로도 개발하기도 한다. 이 방법은 **middle-out** 방식이라고 한다. 어떤 feature를 위해서 도메인에서 어떤 책임이 필요한지를 고민한다. 먼저 도메인 객체를 만든 후에 layer를 올라가면서 개발한다. 이 방식으로 진행하면 test double의 사용을 최소화할 수 있다. 특히, stub의 사용을 최소화 할 수 있다. 또한 이 방식은 도메인 로직에 집중하게 되어서 도메인 로직이 다른 계층으로 흘러들어가는 것을 방지할 수 있다.
- Classicist와 Mockist 방식 모두 layer->layer로 개발하는 방법보다 feature->feature로 구성하는 방법을 선호한다.

#### Fixture setup

- Classicist 방식은 SUT(테스트 대상)을 위해서 모든 collaborator를 개발해야 하는 책임이 있다. 이때, 직접적인 collaborator외에도 다른 간접적인 객체들도 생성을 해야하는 경우가 발생할 수 있다. Mockist들은 반대로 SUT와 직접적으로 관련된 collaborator들만을 mock을 이용해서 생성하면 되기 때문에 복잡한 fixture를 만들 필요가 없어진다.
- 현실에서는 classicist들은 fixture 메서드등을 이용하여 복잡한 fixture들을 재사용하려는 경향이 있다. 또한, 어떤 객체는 생성하는 데에 많은 비용이 들 수도 있다.

#### Test isolation

- Mockist style에서 코드 변화로 인해 버그가 생기는 경우에는 관련된 테스트 하나만 실패한다. Classicist의 경우에는 해당 클래스와 관련된 다른 클래스들의 테스트들도 실패하게 된다. Mockist들은 이러한 이유로 자신들의 방식이 더 좋다고 말한다. 그러나 Classicist들은 그렇게 생각하지 않는다. 보통은 문제점을 찾고 그것 때문에 다른 테스트가 깨지는 것이라는 것을 확인하기 쉽기 때문이다. 그리고 보통은 수정을 하고 테스트를 진행하게 되면 어떤 수정 부분이 문제를 발생시키는지 알 수 있기 때문에 테스트를 고치기 쉽다.
- test 크기도 중요하다. Classicist들은 하나의 객체를 테스트할 때 다른 여러 실제 객체들을 사용하기 때문에 하나의 테스트가 하나의 객체가 아닌 하나의 객체 그룹을 테스트한다고 생각하게 된다. 즉, 해당 그룹이 매우 커지게 되면 어떤 부분에서 버그가 발생하는지 찾기 어려울 수 있다. 테스트의 크기가 너무 큰 것이다. Mockist들은 하나의 객체와 다른 모든 mock을 테스트 하기 때문에 테스트의 크기가 너무 커지면서 발생될 수 있는 고통을 경감할 수 있다. 가장 좋은 방법은 클래스마다 fine-grained test를 생성하는 것이다. Cluster는 합리적으로 보이지만 매우 적은 경우에서만 사용되어야 한다. 만약 너가 어떤 coarse-grained test로부터 버그 부분을 찾는 것 때문에 고통 받는다면 더 작은 테스트를 개발해서 test의 크기를 작게 해야한다.
- Classicist의 테스트는 작은 통합테스트라고 불릴 정도로 다른 객체들과의 상호작용을 검증한다. Mockist 방식의 테스트는 이러한 상호작용을 테스트 하지 못한다. 통합테스트와 E2E테스트가 강력한 버그 방지 기능을 제공한다는 것을 이해한다면 Classicist의 테스트가 Mockist방식보다 더 나은 버그 방지 기능을 제공한다는 것을 알 수 있다.

#### coupling tests to implementations

- Classicist 방식의 테스트는 오직 input에 대한 output이나 객체의 상태와 같은 마지막 결과만을 고려한다. 반면에 Mockist는 다른 협력자들과 의도한 상호작용을 하는지를 확인한다. 후자의 경우에는 내부 구현과 과도한 엮임을 형성한다.
- TDD에서는 이러한 결합이 문제를 야기할 수 있다. Mockist 방식의 테스트는 테스트 작성시에 내부 구현에 대한 고민을 해야 한다. Classicist의 테스트는 인터페이스와 결과만을 생각하게 된다. 즉, 내부 구현의 변경이 일어나면 전자의 테스트는 많은 테스트 수정을 해야하지만 후자는 그렇지 않다.

#### Design Style

- 두 방식은 design 결정에도 다른 영향을 미친다.
- Mockist는 outside-in 방법을 사용하기 때문에 domain model에 집중하고자하는 개발자들에게는 맞지 않는다.
- 작은 단계에서 mockist들은 어떤 값을 전달하는 method보다 object를 모으는 method를 선호한다는 것을 발견했다. e.g. 어떤 객체들의 그룹에서 정보를 수집해서 report 문자열을 만드는 동작을 구현한다고 해보자. Mockist들은 보통 문자열을 모으는 string buffer를 parameter로 메서드들에게 제공한다.
- Mockist들은 **Demeter 법칙** 을 만족하기 위해서 *method1().method2().method3()* 와 같은 **train-wreck** 을 피하라고 말한다. 반대로 **middle object**도 문제가 될 수 있다.
- Mockist들은 자신들의 방식이 **Tell Don't Ask** 원칙을 지키는 데에 도움이 된다고 한다.
- Mockist들은 **role interface** 를 선호하고 자신들의 방식이 협력자들을 mock을 이용해서 대체하기 때문에 훨씬 role interface를 만들기 수월하다고 말한다.
- 위의 이유들 때문에 Mockist들은 자신들의 방식을 선호한다. TDD의 시작은 점진적인 변화에도 대응하는 regression testing의 자동화하려는 욕구에서 시작되었다는 것을 기억해야한다.

#### So should I be a classicist or a mockist

- Martin은 classicist이지만 이 질문에 대답하기 어려워 한다.
- Martin이 생각하기로는 대부분의 경우에 어떤 객체에 대해서 생각할 때 그 객체가 해야하는 일에 대해서 생각하지만 어떻게는 크게 중요하지 않다고 한다. 그래서 자신에게 Mockist의 방식은 과도한 구현에 종속되는 테스트를 만들게 된는 과정으로 받아들여진다고 한다.
- 그렇지만 그도 스스로 Mockist 방식으로 toy project 이상의 무엇인가를 해본 경험이 없기 때문에 확실하게 무엇이 좋다고 표현할 수 없다고 한다.

- 너가 원하는 스타일을 해라. 테스트가 깨지는 이유 때문에 테스트를 디버깅하는 데에 많은 시간을 들인다면 Classic TDD를 사용하면 개선된 결과를 얻을 수 있고, object들의 적당한 행동을 정하지 않았다면 mockist 방식이 나은 선택일 수 있다.


- - -

Test-induced design damage or why TDD is so painful
===================

test-induced desing damage
----------------

- [출처](https://enterprisecraftsmanship.com/posts/test-induced-design-damage-or-why-tdd-is-so-painful/)

- TDD는 단순히 test를 작성하는 것이 아니다. TDD는 **test-first approach**가 가장 중요하다.
- 많은 사람들은 TDD와 unit test 작성이 code에 문제를 일으킨다고 생각한다.

- unit test를 위해서는 코드에서 isolation이 필요하다. isolation은 DB와 같은 외부 시스템과 test code 사이에 존재해야한다. 그렇지 않으면 테스트의 속도가 느려지거나 DB에 의해서 테스트가 실패하는 의도치 않은 결과를 얻게 될 수 있다.

- 아래와 같이 테스트를 위해서 코드를 작성한다고 해보자. 보통은 아래와 같은 코드를 많이 작성한다. 특히, Spring을 사용하는 경우에는 많이 봤을 것이다

```
class SampleControllerTest {
  @Autowired
  lateinit controller: SampleController
  @MockBean
  lateinit repo: SampleRepository

  @Test
  fun `test sample controller`(){
    every { repo.findAll() } returns Flux.just(sample1, sample2)
    StepVerifier(controller.getAll())
       .expectNextCount(2)
       .verifiyComplete()
  }
}
```

- 위의 테스트는 많은 이유에 의해서 문제를 발생시킨다. 특히, **false positive**(테스트에 문제가 없는데 문제가 있다고 말하는 경우)를 발생시킨다. 이렇게 되면 개발자는 언젠가는 무시를 하게 된다(@Ignore 를 설정한단다든지, 주석처리한다든지해서). 결국, 테스트로서의 가치를 잃어버리게 되는 것이다.
- 의존성이 많은 코드는 테스트할 때 그 의존성들 때문에 많은 노력을 기울여한다. 많은 사람들이 이러한 의존성을 쉽게 mock처리하는데 이 때 테스트가 깨지기 쉬운 코드가 된다. 외부의존성이 없는 unit-testing은 design damage를 야기하지 않는다.
- 요점은 design damage는 mock남용과 같은 외부 의존성을 제대로 컨트롤하지 못하는 데에서 기인한다고 볼 수 있다.


how to do painless TDD
----------------

- test induecd design damage를 제거하는 방법은 mock을 남용하지 않는 것이다.
- 소프트웨어에서 가장 중요한 것은 business critical code를 테스트할 수 있는 **견고한** test suite
- 만약 mock없이 비즈니스 코드를 테스트 불가능하다면 외부 의존성과 비즈니스 코드를 분리해야한다.
- 외부 의존성과 비즈니스 코어 로직을 분리하면 외부 의존성을 테스트하는 *test suite*에 의존성 설정 코드가 줄어든다. 코드가 줄어든다는 것은 관리 포인트가 줄어든다는 것을 의미한다.
- web MVC의 controller의 경우에는 비즈니스 로직이 없다. 비즈니스 로직이 없는 곳에 단위 테스트를 하는 것은 득보다 실이 더 크다.
- 위의 생각들로부터 우리는 "단위테스트는 오직 외부 의존성이 없고, 비즈니스 로직을 검증하는 경우에만 한다"라는 생각을 할 수 있다.
- unit test가 필요한 이유는 절대로 100% test coverage가 아님을 명심하자. unit testing의 목적은 변화에 깨지지 않고 비즈니스 로직을 검증하는 안정망을 확충하는 것이다.

Integration testing or how to sleep well at nights
-----------------------

- integration test(통합 테스트)는 단위테스트보다 높은 추상화 단계에서 동작한다. 둘의 가장 큰 차이는 unit test와 달리 통합테스트에는 외부 의존성이 있는 테스트라는 것이다.
- 외부 의존성은 2가지로 나뉜다: 개발자가 관리할 수 있는 외부 의존성(ex. 시스템 전용 DB), 관리 불가능하거나 다른 시스템과 동시에 사용하는 외부 의존성(ex. MSA에서 다른 어플)
- 최선의 선택은 unit-testing과 통합 테스트를 함께 사용하는 것이다. 단위테스트만 있으면 실제로 외부 의존성과 상호작용하는 **usecase**를 검증하지 못한다. 저자는 통합테스트로 하나의 *happy* usecase 흐름을 검증하라고 했다. 물론, 비즈니스 로직으로 검증 못하는 경우에도 통합테스트로 검증해야한다.
- 어떤 interface의 모든 method를 구현하는 **stub**를 만들어 테스트에 사용하게 되면 그 인터페이스에 너무 의존하게 되면서 깨지기 쉬운 테스트를 야기한다.

The most important TDD rule
-----------------------

- unit test에서 가장 주요한 생각은 unit test는 그 대상의 추상화 단계에서 딱 한 단계 높은 단계에서 이루어져야한다는 것이다. 이 말의 의미는 테스트할 때는 테스트 대상의 결과를 검증해야하고 절대로 구현(같은 추상화 단계)을 테스트해서는 안된다는 말이다. 어떤 대상의 추상화 단계와 같은 추상화 단계에서의 검증은 깨지기 쉬운 코드가 된다. mock을 사용하는 경우에 많은 상황에서 이러한 원칙을 어긴다.
- 깨지기 쉬운 코드는 리팩토링을 막는 장애물이다.
- 같은 추상화 단계의 테스트는 캡슐화 원칙을 어기게 된다(캡슐화 원칙을 어기는 설계를 하게 만드는 것은 아니다)

Stubs vs Mocks
--------------

- Test double에서 주로 사용하는 Mock과 stub은 약간 다른 개념이다.
  1. Mock: 테스트에서 사용하는 **동적인 외부 의존성**. 어떤 상황에 어떤 행동이나 어떤 결과값을 전달하도록 설정되어 있다.
  2. Stub: 테스트의 의존성 대상의 행동을 모사하도록 **정적**으로 만들어진 테스트 클래스.
- 고전적인 엔터프라이즈 소프트웨어 설계에서는 통합테스트에서는 test double(mock, stub)을 사용하고, 단위 테스트에서는 실제 객체를 사용하도록 권고한다. 다른 소프트웨어에서는 3rd party library같은 경우에는 단위 테스트에서도 test double을 사용한다.

TDD best practices
---------------

#### test first v code first

- test first approach(TDD)
  - 코드에 대한 빠른 피드백을 해준다.
  - 설계에 대한 힌트를 준다. 구현이 생각나지 않는 코드에 대한 fail 테스트를 먼저 작성해서 개발자가 구현에 대한 힌트를 얻을 수 있다.
  - prototype과 같이 빠른 구현이 필요한 경우에 불필요한 작업으로 인한 오버헤드가 있다.
  - 요구사항이 명확한 경우에 쉽게 작성할 수 있다
- code first approach
  - 단위 테스트가 false positive하기 쉽다: 테스트를 코드 작성 다음에 하기 때문에 테스트를 작성할 때 구현을 생각하게 된다.
  - 프로토타입과 같이 빠른 구현이 필요한 경우에 테스트를 작성하지 않고 진행할 수 있다.
  - 요구사항이 명확하지 않거나 여러 실험이 필요한 경우에는 테스트 코드를 작성하지 말아야한다. 대신에 코드를 먼저 작성해보자.


#### test doubles are for external dependencies only

- stub을 사용하는 경우에 꽤 도움이 되지만, 내부의 의존성조차 stub으로 대체하는 개발자들이 있다. 테스트 대상에 대한 고립이 목적인데 응집력있는 그룹을 형성하는 클래스조다 mock이나 stub으로 대체하면 안되다.
- 테스트 대상의 내부 의존성은 꼭 실제 객체를 사용하자.

#### A single unit for unit-testing is an aggregate

- 또 다른 좋은 선택은 클래스가 아닌 aggregate를 단일 SUT(테스트 대상)로 다루는 것이다.

- - -

Focus on Integration Tests Instead of Mock-Based Tests
=======================================

- [원글](https://phauer.com/2019/focus-integration-tests-mock-based-tests/)

TL;DR
-----

- 전통적인 mock-based 단위테스트에서는 모든 클래스를 고립시킨다. 여기에는 다음과 같은 문제점이 존재한다
  - 실제 상호작용을 테스트하지 않는다
  - 리팩토링하기 힘들다
- 대신에 실제 객체와 대상을 사용하는 통합테스트 자체에 집중해라. 이 테스트는 모든 계층을 관통해야한다. 이 테스트의 이익은 다음과 같다
  - 구현이 아닌 결과를 테스트한다
  - 실제 production과 같은 테스트를 하게 된다
  - 높은 test coverage
  - 리팩토링하기 쉽다
- In-memory가 아닌 production DB를 사용하면 더 실제와 비슷한 환경에서 테스트 가능하다. 이 때 production DB로 TestContainer와 같은 것을 사용하면 좋다
- 내가 제시하는 통합테스트는 최적의 테스트 장소이다. 설정하는 노력과 생산성사이에서 좋은 절충안이 될 것이다


Isolated Mock-Based Unit Tests
----


- 가장 전형적인 단위 테스팅은 모든 클래스를 다른 대상(외부 의존성, 다른 클래스 등등) 고립 시키는 것이다. MVC에서 보자면 아래와 같은 형태가 될 것이다.

```
ControllerTest -> Controller -> ServiceMock

ServiceTest -> Servie -> EntityMock
                      -> RepositoryMock

RepositoryTest -> Repository -> In-memory DB
```

- 이 방식에는 다음과 같은 문제점이 있다
  - unreliable tests: 시스템 내의 객체끼리의 상호작용을 검증하지 않았고 약한 가정에 의존했기 때문에 어떤 테스트가 성공했다고 해서 시스템 전체가 테스트한다는 보장이 없다. 실제로 DB를 mock으로 사용하는 경우에 DB에서는 불가능한 query를 하기도 한다. 이를 검증할 수 있는 방법이 사라진다.
  - painful refactoring: mock-based test는 구현에 너무 의존한다. 구현에 의존하게 되면 리팩토링이 힘들어진다.
  - laborious: test suites에 수 많은 테스트코드와 의존성 코드를 설정해야한다
  - In memory DB != production DB: in memory DB를 사용하면 *unreliable tests*에서 발생한 문제점이 발생할 수 있다.

Intergation Tests
-----------

- 아래는 문제점을 개선한 통합테스트 방법이다.
  - 통합테스트에서는 관리 불가능한 외부 의존성을 제외한 모든 대상을 실제 대상을 사용한다.
  - DB는 production과 같은 버전의 DB를 사용한다. 이 때, TestContainers와 같은 docker container를 이용해서 사용하면 좋다.

```
ControllerTest -> Controller -> Service -> Entity
                                        -> Repoistory -> Read DB(Test containers)
```

- 이 방식은 다음과 같은 장점이 있다.
  - Accurate, meaningful and production-conse tests: 실제와 최대한 비슷한 환경에서 테스트하기 때문에 실제에서 발생할 수 있는 많은 버그를 최소화할 수 있다. 이는 자신감으로 이어진다
  - Robust against refactorings: 통합테스트에서의 구현에 대한 의존성을 줄여서 리팩토링시에서 구현이 변경되더라도 테스트가 깨지지 않는다.
  - one test class to write: 이상적으로는 하나의 통합 테스트를 작성하는 것이 좋다. 처음에 DB 설정등이 어려울 수 있지만, 한 번 해놓은 설정으로 계속 사용할 수 있기 때문에 오히려 경제적이다.

- 어떤 대상을 검증할 때는 행동의 결과만 테스트해야한다. 구현은 신경쓰지말자.
  - 어떤 대상이 실제로 exception을 던지는지 테스트해야할까? 아니다, 더 중요한 것은 이 error 이루어지는 동작이다. 예를 들어, 어떤 예외가 발생하고 그 예외가 발생한 것이 아니라 그 예외를 적절하게 사용자에게 전달하는 것이 중요하다.

Challenges and Reservations
----------------------

- 위의 테스트 방법론에는 여러 반론이 제기될 수 있다

#### Execution Speed

- mock대신 실제 객체를 사용하는 경우에는 테스트 속도가 문제가 되지 않는다. 문제가 될 수 있는 부분은 DB와 같은 외부 의존성을 실제로 사용하는 경우와 DI framework에서 발생할 수 있는 오버헤드이다.

1. DI framwork
  - 스프링과 같은 DI framework의 속도는 매우 느리다
  - 속도가 신경쓰인다면 DI framework를 사용하기보다 실제로 주입해서 사용하기를 추천한다.
  - 실제로 객체를 주입하는 경우에는 DI 자체를 테스트하지 못하는 문제점이 있다. 그러나 대부분의 경우에는 DI보다는 로직을 테스트하기 위해서 테스트를 작성하기 때문에 문제가 되지는 않는다
2. Container 설정 시간
  - container를 up하는 시간은 피할 수 없다. 성능과 속도사이의 trade-off라고 생각해라.
  - test 사이에 up-down을 반복하지말고 test 한 번 할때마다 docker-compose와 같은 방법으로 하나의 test db를 설정해서 사용하면 속도가 조금 줄어들 것이다.

#### More Effort for Input Data Creation and Output Data Assertions

- 실제 input data를 만들고 결과를 검증하는 것이 힘들다고 말하는 경우도 있다. 실제로 맞는 말이다. 이런 경우에 다음과 같은 해결책이 있다
  - hepler function을 사용하자. factory pattern를 이용해서 객체를 생성하는 수고를 줄이도록 하자.
  - parameterized test를 하자. 같은 경우에 비슷한 테스트를 여러 번 작성하는 수고를 줄일 것이다

#### Overkill for Coner Case Testing

- 어떤 corner case를 테스트하기 어렵다는 의견이 있다. 예를 들어, DB에 string으로 들어가야할 곳에 int를 넣는다든지하는 코너 케이스이다.
- 통합테스트에서 원하는 코너케이스를 테스트를 작성하자. 이미 통합 테스트가 구축되어 있다면 단순히 테스트를 추가하는 것은 어렵지 않다
- 만약 통합테스트를 통해서 원하는 코너 케이스를 검증하는 테스트를 작성하기 어렵다면 실제 일어나지 않는 경우일 수도 있다. 이러한 경우의 수를 테스트하기 위해 노력하는 것은 불필요하다.

#### Untestable Features and Conditions

- 조건 Y에서 feature X를 테스트하기 힘들다는 의견이 있을 수 있다. 특히, 결과만을 테스트하게 되면 내부의 동작은 검증 못하지 않느냐는 것이다.
- 어떤 대상을 검증하기 위해서 굳이 그 검증 자체를 테스트할 필요는 없다. 어떤 결과든지 일련의 과정을 통해서 검증되는데 이 때 일련의 과정에서 원하는 결과가 검증되는 것이다. ex) DB와의 인터넷 연결을 검증할 필요 없다. DB에서 데이터가 제대로 왔다면 검증된 것이다.

#### We Already Have VM-Based System Tests

- VM으로 이미 test 환경이 구축되어 있어도 이는 깨지기 쉬운 환경이다. 작은 실수 하나만으로도 VM test 환경이 깨질 것이고 이를 고치기 위한 수고가 들것이다.





- - -

참고 자료
========

- [테스트 주도 개발](http://www.yes24.com/Product/Goods/12246033)
- [테스트 주도 개발로 배우는 객체지향 설계와 실천](http://www.yes24.com/Product/Goods/9008455)
- http://codemanship.co.uk/parlezuml/blog/?postid=987
- https://martinfowler.com/articles/mocksArentStubs.html
- https://team-agile.com/2021/02/06/chicago-and-london-style-tdd/
- https://khalilstemmler.com/wiki/test-doubles/
- https://phauer.com/2019/focus-integration-tests-mock-based-tests/
- https://enterprisecraftsmanship.com/posts/test-induced-design-damage-or-why-tdd-is-so-painful/  
- https://blog.ncrunch.net/post/london-tdd-vs-detroit-tdd.aspx
- https://dhh.dk/2014/test-induced-design-damage.html
- https://martinfowler.com/articles/is-tdd-dead/
- https://medium.com/@adrianbooth/test-driven-development-wars-detroit-vs-london-classicist-vs-mockist-9956c78ae95f
