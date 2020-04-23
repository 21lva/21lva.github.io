---
layout: post
title:  "소개"
date:   2019-12-18 01:02:59
name: k8s_basic.md
category: kubernetes
---

- 컨테이너 오케스트레이션 : 여러 개의 서버에서 컨테이너를 배포하고 운영

k8s
===

- [참고자료](https://blog.2dal.com/2018/02/28/kubernetes-intro/)
- k8s(kubernetes)는 **오케스트레이션** 도구이다(like docker swarm)
- 컨테이너를 쉽고 빠르게 배포, 확장해주는 오픈소스

- - -

특징
======


- automatic binpacking : 컨테이너가 서버(호스트)의 물리적 리소스를 충분히 활용할 수 있도록 자동으로 배치하는 것.
- self-healing : 운영자가 기대하는 상태를 유지하는 것.
- horizontal scaling : app이 제공하는 기준으로 ReplicaSet을 scailing
- Automated rollouts and rollbacks : 변경사항을 전체 인스턴스 중단이 아닌 점진적으로 변경사항 적용
- Secret and configuration management : Secret key, Configuation을 관리하기 위해 Secret, ConfigMaps Object가 제공되기 때문에 이미지의 변경없이 키/설정을 업데이트할 수 있고, 이를 외부에 노출(expose)하지 않고 관리/사용할 수 있다. Secret은 데이터를 binary 형태로, ConfigMap은 text 형태로 저장한다.
- Storage orchestration : local storage를 비롯해서 Public Cloud(GCP, AWS), Network storage등을 구미에 맞게 자동 mount할 수 있다.
- Batch Execution : batch, CI 작업의 수행을 관리할 수 있다. 원한다면 배치실행 실패 시 container를 교체하는 것도 적용 가능하다.
- 다양한 배포 방식 제공 : cronjob, Deployment, Job 등등
- Ingress
- namespace, label

- - -

구조
=====

- Master + Node(Worker, minion) 구조

Master
------

- worker들을 관리하는 역할
- **API server, Controller manager, Schduler, etcd** 로 구성

#### 1) API server

- k8s에서 동작하는 모든 형태의 요청과 응답(HTTP/HTTPS rest api)를 통신해주는 역할
- 모든 컴포넌트는 API server를 지켜보다가 자신과 관련 작업을 수행
- kubectl등의 작업도 여기서 수행

#### 2) controller manager

- ReplicaSet, Deployment 같은 controller들의 Deamon형태를 포함
- Controller들을 생성 배포하는 역할

#### 3) Scheduler

- 각 리소스(Pod, Service)들을 node에 배분하는 역할

#### 4) etcd

- 설정값, 클러스터의 상태 등을 저장
- key-value 형태의 저장소
- 안정성을 위해 보통은 여러개의 etcd 프로세스 생성

Node
----

#### 1) Kubelet

- node마다 하나씩 존재하는 것
- master의 api server와 통신
- Service, Pod 을 실행, 중지, 상태 유지

#### 2) cAdvisor

- 동작중인 컨테이너의 리소스 사용량 등을 체크하여 API server에 전달

#### 3) Kube-Proxy

- 노드로 들어오는 트래픽을 적절히 컨테이너로 라우팅하는 것. 일종의 로드밸런서
- 외부에서 들어온 요청을 다른 node에 있는 pod에 전달하는 역할

#### 4) Pod

- 가장 작은 배포 단위(컨테이너가 아님)

- - -

Object
======

- k8s의 가장 기본적인 단위
- controller는 오브젝트를 관리
- **Spec** : 사용자가 기대하는 상태. yaml, json 등으로 정의되어 있음
- **Status** : 현재 상태
- 기본 오브젝트 : Pod(컨테이너화된 어플리케이션), Volume(디스크), Namespace(패키지), Service 등이 존재

Pod
----

- 가장 기본적인 배포 단위
- 1개 이상의 컨테이너를 포함한다. 보통은 1개
- 1개의 Pod은 1개의 node위에서 실행
- 동일한 작업(spec)을 하는 Pod들은 컨트롤러에 의해 여러 Node에 분배
- 같은 Pod내의 컨테이너, volume등은 서로 IP를 공유하기 때문에 **localhost** 로 통신 가능

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app:app1
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 8080
```
- 사항
  - apiVersion : k8s의 api version
  - kind : 종류
  - metatdata.name : pod이름
  - spec : 원하는 상태


Volume
------

- 컨테이너 내부에서 실행하는 디스크는 컨테이너가 사라지면 같이 사라짐(유실)
- 이 때, 컨테이너 외부에서 사용하는 디스크가 volume object. Pod이 마운트해서 사용
- Pod내의 컨테이너들은 같은 Volume을 공유

Service
-------

- 여러개의 Pod을 생산, 분배, 재시작, 로드 밸런싱 등을 수행
- label을 통해서 특정 Pod들만 사용

```yaml
kind: Service
apiVersion: v1
metadata:
  name: service1
spec:
  selector:
    app: app1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

- port : 클러스터 내부에서 사용하는 port. 여기로 전달된 요청은 targetPort로 전달된
- targetPort : 실제로 사용하는 port. 트래픽을 수신하는 port


위의 Pod을 이용해서 서비스와 팟을 만들어보자

```yml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
  labels:
    app: app1
spec:
  containers:
  - name: hello-world
    image: hello-world
    ports:
    - containerPort: 8090

---

kind: Service
apiVersion: v1
metadata:
  name: service-hw
spec:
  selector:
    app: app1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

조심하자. yml은 **tab** 대신 **space** 를 써야한다

```
//pods, service 확인
kubectl get pods
kubectl get service

//pod, service 생성
kubectl apply -f a.yml
```

- 위의 과정은 다음과 같은 순서로 진행된다
  1. kubectl에서 api server로 Pod 생성을 요청
  2. api server에서 etcd로 node에 할당되지 않은 pod이 있음을 알림
  3. scheduler가 etcd 변경사항을 확인하고(api server통해서) node에 pod 생성
  4. 해당 node의 kubelet에서 pod의 정보를 api server에 지속적으로 보고
  5. api server에서 받은 pod정보를 etcd에 전달  

Namespace
-----------

- k8s의 논리적 분리 단위
- Pod, Service는 namespace별로 생성 관리 가능
- 하나의 클러스터에 여러 환경(개발,운영 등)을 관리 가능하게 함
- namespace별로 사용자의 권한을 관리 가능
- 논리적 분리단위이기 때문에 다른 namespace의 pod과 통신은 가능

label
-----

- k8s의 리소스들이 가질 수 있는 조건
- 서비스에서 특정 label을 가지는 pod들을 묶어서 배포하는 형식으로 사용 가능
- 복수의 라벨을 가질 수 있음(key-value형태)

- - -

Controller
=========

- 오브젝트를 배포하는 데에 사용
- Replication Controller, Replication Set, DaemonSet, Job, Cronjob, StatefulSet, Deployment 등이 존재

Replication Controller
--------------------

- 지정된 숫자의 Pod을 유지하고 관리
- 설정 사항
  - Selector : 가져올 pod들의 label 기술
  - Replicas : 유지되어야 하는 Pod의 수
  - template : Pod에 대한 정보가 들어 있음. template의 정보가 다르면 RC가 나중에 동작하더라도 이전의 pod은 삭제하지 않음(template에 맞는 pod들만 다룬다는 얘기)

ReplicaSet
----------

- Replication controller의 업그레이드 버전
- equality 기반의 selector가 아닌 set기반의 selector 사용

Deployment
------------

- stateful app을 배포할때 주로 사용하는 컨트롤러
- **롤링업그레이드** 같은 새로운 기능 추가

- - -

참고자료
==========
- <https://bcho.tistory.com/1257?category=731548>
- <https://blog.2dal.com/2018/03/28/kubernetes-01-pod/>
- <https://arisu1000.tistory.com/27829?category=787056>
- [쿠버네티스 공식 문서](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)
