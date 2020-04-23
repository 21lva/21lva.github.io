---
layout: post
title:  "패키지"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_package.md
category: java
---

패키지
=====

- 각 클래스들이 들어 있는 디렉토리를 의미
- 기본적인 자바 라이브러리도 java.lang, java.util 등의 패키지로 되어 있다

패키지 만들기
----------

```java
pakcage a.b.c;


public class d{

}
```

- src/a/b/c 밑의 d.java 파일에 저장되어 있다
- 패키지에 속하지 않은 클래스는 **default package** 에 속하게 된다.

서브 패키지
----------

- 위의 디렉토리 src/a/b/c 밑에 f라는 디렉토리가 있고 여기에 e.java가 존재하면 서브패키지가 만들어진 것
- 하위 패키지는 상위 패키지에 종속된다

패키지 사용하기
-----------

```java
package 패키지경로

import a.b.c.d;//a.b.c 패키지의 d 클래스 사용
import a.b.c.*;//a.b.c 패키지의 모든 클래스 사용. 하위 패키지도 사용
```

- java파일의 퍼블릭 클래스 외에는 사용할 수 없다.[관련내용](https://stackoverflow.com/questions/26802795/java-importing-custom-classes-which-are-not-public)
