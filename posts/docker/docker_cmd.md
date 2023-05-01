---
layout: post
title:  "docker 1 - 명령어 & 이미지"
date:   2021-08-22 01:02:59
author: inhyuk
category: docker
tags: docker
cover:  "/assets/instacode.png"
name: docker_cmd.md
---


Dockerfile 명령어
========

- Dockerfile을 작성하기 위한 명령어를 설명한다.

FROM
------

- 기반 이미지 설정

LABEL
------

- 컨테이너 이미지의 메타 데이터를 적시.
- key-value형식

USER
------

- 명령어 실행 계정


WORKDIR
------

- 명령어 실행 작업 디렉토리 설정
- 디렉토리가 없을 경우 생성

EXPOSE
------

- 컨테이너를 실행 시 Listen할 port 설정

COPY vs ADD
------

- 파일을 도커 이미지로 복사하는 명령어
- COPY: 단순 복사만을 지원
- ADD: 복사 기능 지원 + 2가지 기능 더 제공
  - 로컬 파일이나 디렉토리 대신 URL 사용가능
  - tar파일 같은 압축 파일을 해제하여 복사

RUN
------

- 이미지 생성시 명령어 실행
- 이미지 생성은 이전 이미지에 하나씩 레이어를 늘리는 과정이다. RUN은 이전 단계의 이미지로 컨테이너를 만들고 그 컨테이너에서 원하는 명령을 실행하고 새로운 이미지를 생성하는(docker commit) 동작이다
- 아래의 ENTRYPOINT, CMD와 달리 이미지 생성시에 실행된다

ENTRYPOINT vs CMD
------

- 이미지로 컨테이너를 생성시에 실행하는 명령어를 작성하게 해줌
- ENTRYPOINT는 항상 적시된 명령어가 실행된다

```Dockerfile
FROM ubuntu

ENTRYPOINT echo hi
```

```
docker build -t name .

//아래의 두 명령어는 같은 결과: hi
docker run --name test test
docker run --name test test echo hello
```

- CMD는 명령어 실행시 해당 명령어를 변경 가능하다.


```Dockerfile
FROM ubuntu

CMD ["echo", "hi"]
#위의 명령어는 다른 명령어로 대체 가능하다
```

```
docker build -t name .

//아래의 두 명령어는 다른 결과를 보여준다
docker run --name test test
docker run --name test test echo hello
```

- - -

Images
======

- 도커 이미지의 구조를 설명한다.

이미지 저장 장소(로컬)
------

- 로컬에서 이미지는 따로 지정하지 않았다면 **/var/lib/docker/**에 저장된다

![Docker image 1]({{site.baseurl}}/post_img/{{page.name}}/docker_image.png)


Docker 이미지 저장소 변경
-------------------

- 도커는 **/** 아래에 저장하기 때문에 이미지가 많아지면 문제가 생길 수 있다.
- 물론 이 저장소를 바꿀 수 있다.

#### 1) System 파일 수정

- Systemd의 데몬 파일을 수정한다. 아래의 **--data-root**를 추가한다

```
[Service]
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/root/dims/
```

- 아래와 같이 바뀐 곳에 image가 저장되는 것을 볼 수 있다.

![Docker image 2]({{site.baseurl}}/post_img/{{page.name}}/docker-image2.png)

#### 2) Symbolic link 사용

```
ln -s /root/dims2/ /var/lib/docker
```

![Docker image 3]({{site.baseurl}}/post_img/{{page.name}}/docker-image3.png)

이미지의 Layer
---------

- 도커 이미지는 여러 layer로 되어있다.
- 예전에는 이미지와 레이어가 같았으나 현재는 이미지와 레이어는 다르다
- 도커 이미지를 pull할 때 변경된 layer만을 다운 받는다.

![Docker image 4]({{site.baseurl}}/post_img/{{page.name}}/docker-image4.png)

- 위의 사진에서 docker는 여러 layer로 되어있고(여기서는 하나지만), 각 layer는 하나의 폴더로써 존재한다.

![Docker image 5]({{site.baseurl}}/post_img/{{page.name}}/docker-image5.png)

- **cache-id**는 layer의 id와 달리 고정되어 있는 값이 아니다.
- layer 하나의 정보는 **overlay2/**밑에 존재한다. 이 폴더를 확인하면 하나의 레이어에 실제 저장된 데이터를 알 수 있다.

Docker's UFS
-----

![Docker image 6]({{site.baseurl}}/post_img/{{page.name}}/sharing-layers)

- UFS(Union file system): 복수의 파일 시스템을 하나의 파일 시스템으로 마운트하는 기능. 여러 파일 시스템에 중복되는 파일이 있으면 가장 나중(위)의 파일로 덮어 씌운다.
- 도커의 layer는 UFS에서의 하나의 시스템에 해당한다.
- 도커 컨테이너는 도커 이미지로 형성되며, CoW(copy on write)로 이루어지기 때문에 컨테이너가 이미지에 영향을 미치지 않고, 상위 레이어는 하위 레이어에 어떤 영향도 주지 않는다.
  - CoW: 하위 레이어에 대한 쓰기 작업 수행시 하위 layer의 데이터를 상위 layer에 복사하여 수행한다. 이 방법은 여러 컨테이너를 만들더라도 공유하는 만큼의 스토리지 절약을 할 수 있다.

- 도커 컨테이너는 Image layer와 Container layer로 구분된다


- Container layer
  - 쓰기 가능한 최상단 레이어
  - 컨테이너가 생성된 후 이루어지는 모든 쓰기 작업은 이 레이어에서 이루어진다.
  - 하위 레이어의 파일에 쓰기 작업을 할 시에 이 레이어에 파일을 복사하여 수정을 한다.
  - 쓰기 및 읽기 속도가 낮다  
  - 쓰기 작업이 많이 이루어지면 데이터가 중복이 이루어지고 복사 작업에 많은 자원이 소모 된다(그렇기 때문에 쓰기 작업이 많이 일어나는 데이터는 container에 저장하지 말고 volume을 이용해야 한다.)
- Image layer
  - 컨테이너 레이어를 제외한 하위 레이어들
  - 오직 읽기만 가능하다
  - 다른 컨테이너와 공유한다.

- 컨테이너의 크기는 Container layer의 크기(Size) + Image layer의 크기(Virtual size - Size)이다. 모든 Container는 Image layer를 공유하기 때문에 단순히 Virtual size로 container의 크기를 가늠해서는 안된다.

Storage Driver
--------------

- storage driver: container 내에서의 파일 I/O 처리를 담당
- **docker info | grep Storage** 명령어로 스토리지 드라이버를 확인 할 수 있고, UFS 종류를 확인할 수 있다.
- volume container의 경우에는 storage driver를 사용하지 않고 직접적으로 host의 file system에 접근한다.


![Docker image 7]({{site.baseurl}}/post_img/{{page.name}}/docker-image6)

- docker container에 접근(**docker exec -it <container id> /bin/bash**)해서 파일을 하나 만들었다. 이 파일은 host의 특정 장소에 위치한다

![Docker image 8]({{site.baseurl}}/post_img/{{page.name}}/docker-image7)

- 아래와 같이 실제로 해당 위치에 파일이 존재한다

![Docker image 9]({{site.baseurl}}/post_img/{{page.name}}/docker-image8)

- 변경 내용들은 **upper dir**에 위치한다
- overlay filesystem에서는 여러 개 디렉토리가 존재한다
  - upper dir: 최상위 디렉토리
  - lower dir: upper dir 밑의 디렉토리. 오른쪽부터 차례대로 쌓아진다.
  - merge: 모든 디렉토리가 겹쳐서 보여주는 디렉토리. 실제 작업은 merge에서 이루어지고 변화만 upper dir에 저장된다

![Docker image 10]({{site.baseurl}}/post_img/{{page.name}}/docker-image9)

- backing filesystem: storage driver가 사용하는 filesystem. **/var/lib/docker** 에 위치해있다.

docker commit & diff
--------

- **docker diff <container id>**: container의 변경 사항을 확인(A: 추가된 항목, C: 변경된 항목, D: 삭제된 항목)

![Docker image 10]({{site.baseurl}}/post_img/{{page.name}}/docker-image10)

- **docker commit <container id> <new image name>**: 현재 container 상태로 이미지를 생성

![Docker image 10]({{site.baseurl}}/post_img/{{page.name}}/docker-image11)

- 위 처럼 새로운 이미지를 생성할 수 있다

![Docker image 10]({{site.baseurl}}/post_img/{{page.name}}/docker-image12)

- 새로운 이미지로 만든 컨테이너에는 기존에 생성한 파일은 존재하지만 **docker diff**에는 나타나지 않는다. 새로운 이미지는 해당 파일이 추가된 상태로 만들어졌기 때문이다.





- - -

참고자료
======

- https://bluese05.tistory.com/77
- https://www.44bits.io/ko/post/how-docker-image-work#%ED%92%80-%EB%B0%9B%EC%9D%80-%EB%8F%84%EC%BB%A4-%EC%9D%B4%EB%AF%B8%EC%A7%80%EB%8A%94-%EC%96%B4%EB%94%94%EC%97%90-%EC%A0%80%EC%9E%A5%EB%90%98%EB%82%98%EC%9A%94
- https://tiqndjd12.tistory.com/60
- https://devaom.tistory.com/5
- https://dark0096.github.io/docker/2018/08/30/docker-storage-driver.html
- https://docs.docker.com/storage/storagedriver/
