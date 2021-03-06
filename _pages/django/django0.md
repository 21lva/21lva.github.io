---
layout: post
title:  "Django 0 - overview"
date:   2019-11-23 01:02:59
author: Inhyuk
category: Django
tags: django
cover:  "/assets/instacode.png"
name: django0.md
---

**장고 DJango** 를 공부해보자.
이 포스트는 [장고 개념정리](https://www.youtube.com/watch?v=LYmZB5IIwAI&feature=youtu.be)와 [장고로 배우는 쉽고 빠른 웹개발](https://www.coupang.com/vp/products/126353394?itemId=373219182&vendorItemId=3904470030&src=1042503&spec=10304968&addtag=400&ctag=126353394&lptag=GOOGLE_SHOPPING&itime=20191124082148&pageType=PRODUCT&pageValue=126353394&wPcid=15745513089490051959508&wRef=&wTime=20191124082148&redirect=landing&isAddedCart=)를 보고 공부한 기초적이 내용이다.

- - -

웹 프로그래밍
============

- 정적 페이지 vs 동적 페이지 : 정적 페이지 언제 어디서 누가 요청을 하던 같은 내용을 보여준다. 동적 페이지는 이와 다르게 다른 내용을 보여준다. 동적 페이지에는 프로그래밍 코드가 포함되어 있어서 페이지 요청 시 HTML문장을 생성해서 보여준다.
- 웹어플리케이션 : 웹 서버로부터 요청을 받아서 DB 수정과 같은 동작을 진행한 후에 결과를 서버에 전달해주는 역할.

- - -

DJango
======

- 가장 많이 쓰이는 파이썬 웹프레임워크

설치
----

#### Django

- 여기서는 리눅스를 기준으로 설명한다.
- 설치는 *pip* 를 이용하는 것이 일반적이다.

```console
sudo pip install Django
//sudo로 하지 않으면 django-admin을 찾지 못할 수가 있다
```

- 프로젝트 : 웹사이트 전체를 의미
- 어플리케이션 : 기능적으로 모듈화된 작은 단위

<!--
Apache2
---------

- 장고에서 제공해주는 서버를 사용해도 상관 없지만,
-->


- - -

MVC, MTV
===========

- **Django** 는 MTV로 구성된다.
- Model : 데이터 베이스에 억세스하는 기능
- Template : 데이터를 유저에게 보여주는 기능. MVC의 View에 해당
- View : 사용자의 입력과 이벤트에 반응하여 Model과 View에 영향을 미침. MVC의 Control에 해당.
- 장고의 경우 View대신 Template, Control대신 View라고 한다(왜 헷갈리게.....)

![MTV]({{site.baseurl}}/post_img/{{page.name}}/mtv.png)

- 웹클라이언트로부터 요청이 들어오면
1. 클라이언트로부터 받은 요청을 URLconf모듈을 이용하여 url을 분석
2. 분석 결과를 이용하여 해당 url의 View를 실행
3. View는 로직을 실행하면서 필요한 경우 Model를 호출한다
4. Model은 처리 결과를 View에 전달한다
5. View는 Template를 이용하여 HTML파일을 생성하고 전송한다


Model
-----

- django의 경우 ORM을 이용한다.
- 객체 관계 매핑(ORM)
1. 데이터베이스와 데이터 모델 클래스를 연결해주는 다리
2. SQL문을 이용하지 않고도 쉽게 DB의 테이블을 조각 가능.
3. 하나의 클래스는 하나의 DB에 해당한다.

- Person이라는 테이블을 만들기 위해서는 Person이라는 클래스를 만들어야한다.
- 해당 클래스들은 models.py에 정의한다.

```py
from django.db import models

class Person(models.Model):
  first_name = models.CharField(max_length=40)
  last_name = models.CharField(max_length=40)
```

Template
-------

- 화면 UI 설계를 하는 부분이다.
- 디자이너는 프로그램 로직을 생각하지 않고도 이 템플릿만 수정하면 된다(협업이 더 수월해짐)
- 개발자가 *html*(템플릿) 파일을 생성하면 장고(특히 View)는 이 파일의 python부분을 해석해서 최종 *html* 파일을 생섷아고 이를 클라이언트에게 전송한다.
- 모든 *html* 파일은 적절한 위치에 있어야만 Django가 사용할 수 있다.
- Django는 필요한 *html* 을 settings.py에 저장되어 있는 TEMPLATE_DIRS 혹은 INSTALLED_APPS에서 찾는다.

URLconf
------

- 클라이언트로부터 받은 url을 분석한다.
- urls.py에 url과 view를 매핑하도록 한다.

```py
from django.urls import path

form . import views

urlpatterns = [
  path('article/2003',views.special)
  path('article/<int:year>',views.year)
]
```
- 만약에 *article/1990* 이 url인 경우에는 views.year(request,year=1990)가 호출된다.

View
----

- URLconf는 url에 맞는 view를 호출한다.
- view는 보통 함수 or 클래스로 정의된다.
- 응답으로는 HTML,redirection,error 메세지 등을 보낸다.
- 보통은 views.py에 정의하지만 다른 곳에 정의해도 무방.

- 아래의 내용은 현재 날짜와 시각을 보내는 view이다.

```py
from django.http import HttpResponse
import datetime

def current_date(request):
  now=datetime.datetime.now()
  html = "<html><body>Now : %s</body></html>"%now
  return HttpResponse(html)
```



- - -

참고자료
=======
(https://unifox.tistory.com/6)
