---
layout: post
title:  "Pod"
date:   2019-12-18 01:02:59
name: k8s_pod.md
category: kubernetes
---

Pod
=====

- k8s에서는 보통 단일 컨테이너가 아닌 컨테이너 여러개(보통은 1개)를 포함하고 있는 Pod를 이용
- k8s안의 컨테이너들은 Pod안의 IP를 공유하기 때문에 서로 포트를 가지고 통신이 가능하다

- - -

LifeCycle of Pod
===========


- pod는 생성에서 삭제까지 lifecycle을 가지고 있음
- **kubectl describe pods <pod이름>** 으로 확인하면 Status가 존재
- Status의 값
  - Pending : pod를 생성한 상태. 내부의 컨테이너가 실행되는 중
  - Running : pod가 돌아가는 상태. 실행중인 pod의 갯수를 확인
  - Succeeded : 컨테이너가 성공적으로 종료된 상태
  - Failed : 종료가 실패한 컨테이너가 있는 상태
  - Unknown

- - -

Health check
=============

- k8s에서는 각 **container**(pod이 아니다)의 상태를 주기적으로 확인하여 문제가 있는 pod을 재시작하거나 서비스에서 제외
- livenessProbe : 컨테이너가 살아있는지 확인. 상태확인이 실패하면 kubelet은 컨테이너를 재시작.
- readinessProbe : 컨테이너가 실행된 후 실제로 서비스 요청에 응답을 할 수 있는지 확인. 실패하면 endpoing controller가 해당 pod가 연결된 모든 service에서 이 pod의 endpoint 정보를 제거(서비스에서 제외된다고 보면된다). pod생성 후 바로 서비스 하는 것이 아닌 별도의 probe를 통해 서비스 전 확인을 한다

prob 방법
--------

#### 1) command probe

- 상태 체크를 shell 명령어를 통해서 수행
- shell 명령어의 결과가 0이면 성공 아니면 실패로 간주

```yml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-app
    image: test-image
    imagePullPolicy: Always
    ports:
    - containerPort: 8080
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
```

*/tmp/headlthy* 가 존재하는지 확인하는 cmd가 있다.

#### 2) HTTP probe

- 가장 많이 사용하는 방식.
- HTTP GET을 이용하여 컨테이너의 상태를 체크

```yml
metadata:
  name: readiness-rc
spec:
  replicas: 2
  selector:
    app: test-app
  template:
    metadata:
      name: test-pod
      labels:
        app: test-app
    spec:
      containers:
      - name: test-container
        image: test-image
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /ppp
            port: 8080
```

path에 Http Get을 보낼 url을 적고, port에는 보낼 port를 적는다

#### 3) TCP probe

- 지정된 port에 TCP 연결을 해서 연결되면 성공 아니면 실패

```yml
apiVersion: v1
kind: Pod
metadata:
  name: test-app
spec:
  containers:
  - name: test-container
    image: test-image
    imagePullPolicy: Always
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5

```

**tcpSocket.port** 에 TCP 연결을 시도. **initialDelaySeconds** 만큼 기다렸다가(pod의 컨테이너가 가동되는 등의 작업에 필요한 시간을 기다려야함) **periodSeconds** 마다 주기적으로 시도

- - -

init container
==============


- app container가 실행되기 전에 pod를 초기화하는 역할
- 여러개의 init container가 존재하면, 차례대로 실행. 중간에 실패를 하면 성공할 때까지 재실행
- pod가 준비되기 전에 실행하는 container라 readinessProbe가 존재하지 않음

```yml
initContainers:
- name: init-myservice
  image: busybox
  command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
- name: init-mydb
  image: busybox
  command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```
- - -

참고자료
======

- <https://arisu1000.tistory.com/27829?category=787056>
