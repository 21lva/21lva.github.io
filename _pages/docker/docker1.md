---
layout: post
title:  "Docker 1 - 기본 서버 올리기"
date:   2019-12-18 01:02:59
author: Inhyuk
categories: Docker
tags: docker
cover:  "/assets/instacode.png"
name: docker1.md
---


Docker-compose를 이용하여 간단한 Apache 서버 생성
======================================

- 먼저 도커를 이용하여 아파치 이미지를 갖는 컨테이너를 생성한다

```console
$ sudo docker run -d --name apacheExercise -p 8080:80 httpd
```

- -d : background로 실행
- --name : 컨테이너 이름 지정
- -p []호스트 포트:컨테이너 포트] : 호스트 포트와 컨테이너 포트를 연결한다.

- 호스트 포트(8080)에 컨테이너 포트(80) 연결되어 있으니 서버가 잘 돌아가는지 확인해보자.
- 웹브라우저에서 **localhost:8080** 를 입한다.

- - -

Docker-compose.yml을 이용한 nginx + nodejs + mariaDB 서버 생성
=======================================


1) docker-compose.yml 작성
---------------------------

- **docker-compose.yml** 은 여러 컨테이너 혹은 하나의 컨테이너를 작성하는 방식을 적어놓은 파일이다.
- 파이썬 기반으로 작성되어 있다.

- links : 컨테이너 끼리 연결. 컨테이너이름:별명(alias)
- ports : 외부에 노출할 포트 설정. "호스트 포트:외부에 노출할 컨테이너 포트" or "외부에 노출할 컨테이너 포트"
- expose : 컨테이너끼리 통신하기 위해 열어줄 포트
- depends_on : 해당 컨테이너가 실행되기 위해서 선행되어야할 컨테이너
- volumes : 컨테이너에 마운트할 Volume. ro(read only), rw(read & write)

```yml
# docker-compose의 버전을 명시. 버전별로 명령어등의 약간의 차이가 있다.
version: "3"

services:
    nginx:
        # 만들어질 container 이름
        container_name: nginx

        # Dockerfile의 위치
        build: ./nginx

        # 컨테이너 끼리 내부적으로 연결할 때의 alias
        # 예를 들어 A:B 의 경우 이 컨테이너 내에서 B라는 이름으로 A에 연결 할 수 있다.
        # 여기서는 nginx에서 app이라는 도메인을 통해서 app 컨테이너에 접근할 수 있다.
        links:
            - app:app

        # 호스트와 연결할 포트:외부에 노출할 포트(둘다 쓸 경우 " "안에 넣어야함)
        ports:
            - "80:80"
        depends_on:
            - app

    app:
        container_name: app

        #Dockfile의 위치
        build: ./app/

        # 환경변수를 지정할 수 있음.
        environment:
            NODE_ENV: localhost
        expose:
            - 3000
        links:
            - db:app_db
    db:
        image : mariadb:5.5
        volumes:
          - "./data:/var/lib/mysql:rw"
        environment:
          - "MYSQL_DATABASE=hello"
          - "MYSQL_USER=hello"
          - "MYSQL_PASSWORD=hello"
          - "MYSQL_ROOT_PASSWORD=root"
        expose:
            - 3306
        ports:
          - "3306:3306"
```

2) dockerfile 작성 및 conf 파일 작성
---------------------

- 위의 nginx와 app의 build에 사용될 dockerfile을 작성해보다

#### (1) nginx의 dockerfile

```Dockerfile
# nginx/Dockerfile
FROM nginx
ADD nginx.conf /etc/nginx/nginx.conf
```

#### (3) nginx.conf 작성

```conf
# nginx/nginx.conf
worker_processes 4;

events {
    worker_connections 1024;
}

http {
    upstream app {
        server app:3000;
        #app의 3000 port와 연결한다
    }

    server {
        listen 80;

        location / {
            proxy_pass http://app;
        }
    }
}
```

#### (3) app의 dockerfile

```Dockerfile
# Dockerfile
FROM node
MAINTAINER 21lva

RUN mkdir /app
ADD . /app
WORKDIR /app

CMD ["node", "index.js"]
```

#### (4) index.js의 mysql모듈 부분

```js
const connection = mysql.createConnection({ 
    host: 'app_db',//alias 이름 설정 
    user: 'hello', //root는 local에서만 사용 가능
    port: 3306, 
    password: 'hello', 
    database: 'engineer' 
    });

출처: https://donochi.tistory.com/208 [뉴질랜드 다이어리(NZ Diary) 라빠]
```
3) 폴더 구조 및 빌드 & 실행
------------------------

- 현재 폴더 구조는 다음과 같다

```
dockerExercise
├─nginx
│ ├──Dockerfile
│ └──nginx.conf
├─app
│ ├──Dockerfile
│ ├──app.js
│ ├──bin
│ ├──routes
│ ├──public
│ ├──views
│ ├──package.json
├─docker-compsoe.yml
└─data
```

- docker-compose.yml이 있는 폴더로 가서 빌드 후 사용하자

```console
docker-compose build
docker-compose up
```

- localhost:80에 접속하여 잘 작동하는지 확인하자.

![docker_compose]({{site.baseurl}}/post_img/{{page.name}}/docker_compose_up.png)
- - -

Docker image 지우기
------------------

- docker를 사용하다 보면 **<none>** 이라고 쓰여있는 image들이 많이 생성된다.
- 다음 명령어를 쓰면 쉽게 제거할 수 있다

```console
docker image prune --filter="dangling=true"
```


- - -

참고자료
=======

- <https://conservative-vector.tistory.com/m/99>
- <https://ho1234c.github.io/2017/01/31/2017-01-31-docker-nodejs/index.html>
- <https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html>
- <https://blog.nacyot.com/articles/2014-01-27-easy-deploy-with-docker/>
