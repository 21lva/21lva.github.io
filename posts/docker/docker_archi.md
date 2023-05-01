---
layout: post
title:  "Docker - Architecture"
date:   2019-12-18 01:02:59
author: Inhyuk
category: Docker
tags: docker
cover:  "/assets/instacode.png"
name: docker_archi.md
---

![archi]({{site.baseurl}}/post_img/{{page.name}}/architecture)


Docker는 위와 같은 구조로 되어 있다.

- - -

docker 파일
========

- docker CLI를 수행하기 위한 binary

- - -

docker-proxy
==========

- contaienr를 기동할 때 **-p 8080:80**와 같은 옵션을 주면 host의 8080 port로 오는 요청을 80으로 port-forwarding해주는 역할
- 외부에서 요청을 보내면 docker-proxy가 해당 요청의 port를 변경해서 docker0라는 bridge를 통해서 요청을 전달한다. 이 bridge에서 veth을 통해서 적합한 container에게 전달된다

- - -

Docker Daemon
=========

- client, registry, drvier의 작업 분배를 담당
- client에게서 HTTP 요청을 받아서 실행 단위인 job으로 분할
- Http Server: client의 요청을 받아서 적절한 handler를 찾는다. 요청은 Handler를 통해서 다른 모듈에서 실행
- image, container, network, volumen등을 관리

contaienrd
---------

- container의 lifecycle을 관리. client로부터 받은 container관련 요청이 dockerd에게서 전달 받는다
- dockerd와 함께 기본적으로 구동되는 daemon process
- 원래는 dockerd에 포함되어 있었지만 분리되었다.

- - -

container runtime
==========

- container의 실행 환경
- container은 여러 격리 기술이 붙은 프로세스이고, 이 격리 기술들은 원래 LXC, libvirt와 같은 driver에 의해 제공되었다. 추후에 이는 **runC**라는 container runtime으로 변경되었다.
- dockerd에서 containerd로 요청 전달 -> containerd에서 containerd-shim 생성 -> containerd-shim에서 runc을 이용하여 container 생성
- runc는 container 생성후 container를 containerd-shim에 전달하고 종료되기 때문에 runtime daemon이 계속 떠있는 것을 방지할 수 있다
- containerd-shim이 있으면 containerd와 dockerd에서의 장애가 container까지 가는 것을 막아준다

- - -

Docker Client
============

- docker와 통신하기 위해 존재하는 client
- 사용자들은 cli를 통해서 Docker client에게 명령을 전달하고 client는 HTTP로 wrapping해서 daemon과 통신한다.
- 로컬에서는 **/var/run/docker.sock**의 UNIX socket을 통해 dockerd의 api를 호출하고, 원격은 TCP을 이용해서 dockerd에게 명령을 전달

![containerd]({{site.baseurl}}/post_img/{{page.name}}/containerd.png)

- - -

Driver
======

- 3가지로 나뉜다

Graph Drvier
---------

- **/var/lib/docker**에 저장되어 있는 container image등의 정보를 이용하여 layered filesystem 제공
- auts, overlay2등이 존재

execdrvier
-------------

- container의 생성 및 생애주기 관리
- 여러 격리 기능을 이용하여 container을 생성하고 실행
- runtime driver: execdriver에서 container등을 관리하는 pluggable driver. LXC, native driver(runc) 등이 존재

Network driver
-------------

- docker engine등에서 오는 network 제어를 위한 추상화 API를 제공
- 아래의 내용에서의 libnetwork가 이것을 구현한다. 아래의 내용에서의 driver들과는 다른 내용
- 아래의 내용 이후에 추가로 설명한다
- libnetwork: network을 관리하는 API를 제공하고 실제 동작하는 driver들을 위한 pluggable interface제공. 이 driver들이 data plane에서 실제로 통신을 담당한다

- - -

Container Network Model
===============

![cnm]({{site.baseurl}}/post_img/{{page.name}}/cnm.png)

- Container network design spec
- container network을 제공하는 데에 필요한 step정의하고 network driver을 위한 추상화 제공

#### 1) Sandbox

- 격리 **Network stack**
- Network stack: container network interface, routing table, DNS 등
- Sandbox는 linux network namespace을 이용하여 구현된다
- Sandbox에는 다수의 network에 연결된 다수의 endpoint가 포함됨

#### 2) Endpoint

- Sandbox를  Network에 연결하는 역할
- veth pair, open vSwitch 등으로 구현된다
- Endpoint는 하나의 Network에만 속한다

#### 3) Network

- 서로 직접 통신 가능한 Endpoint 그룹
- Linux bridge, VLAN등으로 구현
- 하나의 switch 역할

- 위의 사진에서 가운데의 endpoint 2개는 서로 통신(layer 2에서는)이 불가능하다
- 하나의 host에 sandbox가 다른 두 container가 존재할 수 있다

Libnetwork
----------

- CNM을 구현한 구현체
- Go로 작성된 오픈소스
- network과 관련된 코드가 daemon에 모두 존재하던 시절과는 다르게 CNM 원칙에 따라 구현된 libnetwork가 dockerd의 network 기능을 대체하였다.
- CNM의 3가지 요소에 해당하는 기능에 추가로 service discovery, ingress loadbalancing, network controll & Management plane 구현

- NetworkContoller: libnetwork의 entrypoint를 제공하고, 사용자가 네트워크를 관리할 수 있는 API 제공
- Network: CNM의 Network의 구현체.
- Endpoint: service enpoint를 의미.
- Sandbox: ip address, mac address, routes, DNS 등을 포함하는 객체. Network 객체가 Enpoint 객체를 생성할때 같이 생성된다.

Driver
-------

- Data plane을 구현
- bridge, overlay, macvlan 등의 드라이버가 존재
- 각 드라이버는 담당하는 네트워크 상의 모든 리소스를 생성하고 관리

Single-host bridge network
------------

- 가장 기본적인 형태의 네트워크
- 하나의 호스트에 여러 컨테이너들이 존재하고 그 컨테이너들을 단일 bridge로 연결하는 형태
- 모든 도커 호스트는 이미 하나의 default-host bridge network을 가진다. 이는 호스트에서 **docker0**로 나타나는 kernel bridge에 매핑된다

```bash
root@s17b59a580ca:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
74b70bd584d7        bridge              bridge              local
root@s17b59a580ca:~# docker network inspect bridge | grep name
            "com.docker.network.bridge.name": "docker0",
root@s17b59a580ca:~# ip -br link
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
```

- 새로운 네트워크와 컨테이너를 생성하고 그 컨테이너에 네트워크를 연결하자

```bash
root@s17b59a580ca:~# docker network create -d bridge x1
19f0b4628a38877f618e512a1638d4ab1569fa09931c01c4df68c14dec9e66b7
root@s17b59a580ca:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
74b70bd584d7        bridge              bridge              local
19f0b4628a38        x1                  bridge              local
root@s17b59a580ca:~# docker container run -d --name c1 --network x1 nginx
b771b965c23c2b77a38fd25497ac4d14e89bc2271eea4f405722e975b335f9f9
root@s17b59a580ca:~# docker network inspect x1 --format '{{json .Containers'}}
{"b771b965c23c2b77a38fd25497ac4d14e89bc2271eea4f405722e975b335f9f9":{"Name":"c1",
```

- 새로운 컨테이너를 만들고 network을 연결한 후에 이전에 만든 컨테이너에 접속하면 제대로 된다.

```
root@s17b59a580ca:~# docker container run -d --name c2 --network x1 nginx
fd193c575824d60414f2b51397e1bb66449ac20384c4ddd3d6a079e98929738c
root@s17b59a580ca:~# docker exec -it c2 /bin/bash
root@fd193c575824:/# curl 172.18.0.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

Mulit-host overlay network
-------------

- Overlay 네트워크는 멀티 호스트.
- 도커는 overlay 네트워크를 위한 natvie 드라이버 제공

- - -

Os containers vs Application containers
=======================

[Os vs application containers](https://blog.risingstack.com/operating-system-containers-vs-application-containers/)

- OS container: host의 kernel을 공유하고 user space는 분리하는 가상 환경. VM에 각자의 os를 설치하는 것 처럼 library와 같은 것들은 따로 설치한다. 여러 프로세스가 동작 가능. LXC 등에서 사용하는 방식
- Application containers: 하나의 서비스를 위해서 사용되는 container. 하나의 도커 컨테이너에서는 하나의 프로세스가 동작. Layered filesystem 사용. Docker에서 사용됨.



- - -

참고자료
======

- [참고자료1](http://cloudrain21.com/examination-of-docker-total-architecture)
- [참고자료2](https://www.leafcats.com/146)
- [참고자료3](https://docs.docker.com/get-started/overview/)
- [참고자료4](https://kadensungbincho.tistory.com/105)
- [참고자료5](https://www.docker.com/blog/docker-networking-takes-a-step-in-the-right-direction-2/)
