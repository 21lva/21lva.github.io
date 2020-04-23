---
layout: post
title:  "vs spring mvc : 구조 차이"
name: sb_structure.md
---

spring boot와 spring MVC의 구조 차이
=============================

#### spring MVC

```
app
  - pom.xml
  - src
    - main
      - resource
      - webapp
        - WEB-INF
        - index.jsp
```

#### spring boot

```
app
  - mvnw
  - mvnw.cmd
  - pom.xml
  - src
    - main
      - java
      - resources
    - test
      - java
```
