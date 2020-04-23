---
layout: post
title:  "Django 1 - 기본구조"
date:   2019-11-23 01:02:59
author: Inhyuk
category: Django
tags: django
cover:  "/assets/instacode.png"
name: 2019-11-23-django1.md
---

기본적인 프로젝트 구조
=================


기본 프로젝트 생성
-------------

- 장고를 설치했으면 장고 프로젝트를 만들어보자.

```console
django-admin startproject mysite
```

- 이름이 *polls* 인 새 어플리케이션을 만들어보자

```console
cd mysite
python3 manage.py startapp polls
```

- 아래와 같은 구조가 된다.

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        apps.py
        migrations/
        models.py
        tests.py
        views.py
```

- mysite/ : 가장 바깥 디렉토리, 이름을 변경하여도 무방하다
- manage.py : 다양한 커맨드라인 유틸리티
- mysite : 프로젝트에 쓰이는 질페 내용들이 저장된 곳
- __init__.py : 패키지임을 알려주는 용도
- settings.py : django의 환경을 저장
- urls.py : 프로젝트 레벨의 url 패턴을 정의하는 최상위 URLconf. 각 어플리케이션마다 urls.py가 따로 존재
- wsgi.py : Apache와 같은 웹 서버와으 WSGI 규걱을 연동하기 위한 파일
- admin.py : Admin사이트에 모델 클래스를 등록해주는 파일
- apps,py : 어플리케이션의 설정 클래스 정의하는 파일
- migrations : DB의 변경사항을 관리.
- models.py : 모델 클래스 저으이
- test.py : 단위 테스트용 파일
- views.py : 뷰 함수 정의


설정 파일 변경
------------

#### settings.py

- 프로젝트의 전반적인 사항을 저장한 곳
- DEBUG : 개발 모드(True), 운영모드(False)
- ALLOWED_HOSTS : 서버의 ip나 도메인. 개발 모드의 경우 아무것도 쓰지 않아도 *'localhost', '127.0.0.1'* 로 인식
- 어플리케이션은 모두 설정 파일에 등록되어야 한다.

```py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls',
]
```

- 사용할 DB도 등록해야 한다. 기본적으로 SQLite3를 사용하지만 다른 DB를 사용할 경우 등록을 해주도록 한다.

```py
DATABASES = {
  'default' : {
    'ENGINE' : 'django.db.backends.sqlite3',
    'NAME' : os.path.join(BASE_DIR,'db.sqlite3')
  }
}
```

기본 테이블 설정
--------------

```console
python3 manage.py migrate
```

- migrate : db의 변경사항이 있을 때 이를 반영해주는 명령어
- db 테이블을 만들이 않았어도 이 명령어를 사용하면 사용자, 그룹 등을 만들어 준다.


지금까지 작업 확인
---------------------

- 확인을 위해 웹서버를 동작해보자

```console
python3 manage.py runserver 0.0.0.0:8000
```

- runserver : 테스트용 웹 서버 제공
- 0.0.0.0 : 현재 명령을 실행 중인 서버의 ip주소가 무엇으로 설정되어 있더라고 웹 접속 요청을 받겠다는 의미
- *python3 manage.py runserver* : 디폴트로 127.0.0.1:8000 을 사용
- *python3 manage.py runserver 8888* : 디폴트로 127.0.0.1 에 포트 8888사용
- *python3 manage.py runserver &* : & 명령어를 추가하면 웹서버를 백그라운드로 실행

- 웹브라우저에 127.0.0.1:8000/adim 를 입력하자
- admin 페이지가 나와서 로그인 해야하지만, 아직 관리자(super user)를 만들지 않았기 때문에 로그인이 불가능하다.
- 다음 명령어를 통해 관리자를 생성하자

```console
python3 manage.py createsuperuser
```

- 만들어진 ip,password를 가지고 admin 웹사이트에 접속하면 *Group, User* 테이블이 만들어져 있는 것을 볼 수 있다.
- 두 테이블은 settings.py의 django.contrib.auth 어플리케이션에 이미 등록이 되어 있다.


- - -

Model 만들기
===========

- 모델을 만드는 것은 다음 순서롤 정의된다
1. models.py에 테이블(클래스)를 정의한다
2. admins.py를 통해 models.py에 정의된 클래스를 admin 화면에 보이게 한다
3. *python manage.py makemigrations* : 변경이 필요한 사항을 추출
4. *python manage.py migrate* : 변경사항을 반영
5. *python manage.py runserver* : 전체 사항을 웹서버에서 확인

1) Table 생성
---------

- **polls** 어플리케이션에 **Question** 과 **Choice** 테이블을 정의하자
- 테이블 하나를 클래스로 정의하고, 각 테이블의 column은 클래스의 멤버변수로 정의한다.
- PK(primary key)는 클래스에 지정해주지 않아도 항상 PK를 NotNull + Autoincrement로 테이블명(소문자)를 접두어로 사용하여 자동으로 만든다

```py
#models.py

from django.db import models

class Question(models.Model):
  question_text = models.CharField(max_length=200)
  pub_data = models.DateTimeField("data published")

  def __str__(self):
    return self.question_text

class Choice(models.Model):
  question = models.ForeignKey(Question,on_delete=models.CASCADE)#FK는 다른 테이블의 PK와 연결된다.
  choice_text = models.CharField(max_length=200)
  votes = models.IntegerField(default=0)

  def __str__(self):
    return self.choice_text
```

2) Admin에 테이블 반영
--------------------

- models.py에 정의된 클래스를 반영하려면 admin.py를 수정해야 한다.

```py
#admin.py

from polls.models import Question,Choice
from django.contrib import admin

admin.site.register(Question)
admin.site.register(Choice)

```

3) DB에 변경사항 반영
------------------

- 테이블 생성, 정의 변경 등 DB에 변경이 필요한 사항은 DB에 실제로 반영해주는 작업을 해야한다.
- *makemigrations* : *migrations* 디렉토리에 migration 파일들이 생긴다.
- *migrate* : migraion 파일들을 이용하여 테이블을 생성

- - -

URLconf
=======

- url에 맞게 view를 실행시켜야 한다

```py
#urls.py

from django.contrib import admin
from django.urls import path
from polls import views

urlpatterns =[
    path('admin/',admin.site.urls),
    path('polls/',views.index,name='index'),
    path('polls/<int:question_id>/',views.detail,name='detail'),
    path('polls/<int:question_id>/result/',views.result,name='result'),
    path('polls/<int:question_id>/vote/',views.vote,name='vote'),
]
```

- path(route,view,kwargs,name)
1. route : url 패턴 문자열
2. view : 호출되는 view 함수. view함수(httpRequest 객체, 추출된 사항들)
3. kwargs : url 패턴외에도 추가적으로 전달하고 싶은 사항을 딕셔너리 형태로 전달
4. name : 각 패턴별 이름을 붙인다.

- **Django** 에는 **ROOT_URLCONF** 라는 변수가 정의 된다. 장고는 url을 분석할 때, 이 변수의 파일(urls.py)을 가장 먼저 분석.

- 장고에서는 각 app마다 하나의 urls.py를 작성할 수 있다(더 선호되는 방식)

```py
#urls.py

from django.contrib import admin
from django.urls import path,include

urlpatterns =[
    path('admin/',admin.site.urls),
    path('polls/',include('polls.urls')),
]
```

```py
#polls.urls.py

from django.urls import path
from . import views

app_name = "polls"
urlpatterns =[
    path('',views.index,name='index'),
    path('<int:question_id>/',views.detail,name='detail'),
    path('<int:question_id>/results/',views.results,name='results'),
    path('<int:question_id>/vote/',views.vote,name='vote'),
]
```

- 장고에서는 namespace로 app마다 view들을 관리한다.
- polls의 detail은 polls:detail로 표기한다(app의 namespace를 지정해야한다)
- Regular expression을 사용할 거면 path대신 url함수를 사용한다

- - -

Template & View
============

- Template와 View를 만들기 전에 settings.py의 **Template** 의 **DIR** 를 다음과 같이 수정해야 한다.

```py
'DIR' =[os.path.join(BASE_DIR,'templates')]
```

- 어떤 app의 template함수는 *polls/templates/polls/* 에 저장한다

```html
<!-- polls/templates/polls/index.html-->

{% raw %}
{% if latest_question_list%}
  <ul>
  {% for question in latest_question %}
    <li><a href="{% url 'polls:detail' question.id%}">{{question.question_text}}</a></li>
  {% endfor %}
  </ul>
  {% else %}
    <p>No polls</p>
{% endif %}
{% endraw %}
```

```py
# views.py
from django.shorcuts import render
from polls.models import Question

def index(request):
  latest_question_list = Question.objects.all().order_by('-pub_date')[:5]
  context = ["latest_question_list":latest_question_list]
  return render(request,'polls/index.html',context)
```

- detail 관련 내용도 만들어 보자

```html
<!-- polls/templates/polls/detail.html-->
{% raw %}
<h1>{{question.question_text}}</h1>

<form action = "{% url 'polls:vote' question.id %}" method = 'post'>
{% csrf_token %} #Cross Site Request Forgery 공력을 대비하기 위한 기능
{% for choice in question_choice_set.all %}
  <input tyupe = "radio" name="choice" value="{{choice.id}}"/>{{choice.choice_text}}
{% endfor %}
<input type="submit" value="Vote"/>
</form>
{% endraw %}
```

```py
#views.py
from django.shortcuts import render, get_object_or_404
from polls.models import Question
from django.http import HttpResponseRedirect
from django.urls import reverse

def index(request):
    latest_question_list = Question.objects.all().order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

def detail(request,question_id):
    question=get_object_or_404(Question,pk=question_id)
    return render(request,'polls/detail.html',{'question':question})

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try :
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExit):
        return render(request, 'polls/detail.html', {'question':question})
    else :
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:results', args=(question.id ,)))
```

- HttpResponseRedirect : redirect한다. 타겟 url은 reverse로 생성한다.
- request.POST['choice'] : name이 choice에 해당하는 값
- selected_choice.save() : db에 값을 저장한다
- reverse : url패턴으로 url스트링을 구한다.

- result view와 result template도 만들어보자.
- 다음의 내용을 views.py에 추가하자

```py
def results(request,question_id):
    question = get_object_or_404(Question,pk=question_id)
    return render(request,'polls/result.html',{'question':question})
```

- result.html도 만들어보자

```html
{% raw %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <h1>{{ question.question_text }}</h1>

    <ul>
      {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }} - {{ choice.votes }} vote</li>
      {% endfor %}
    </ul>
    <a href="{% url 'polls:detail' question.id %}">Vote again?</a>
  </body>
</html>
{% endraw %}
```
