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

설계하기
========

-
