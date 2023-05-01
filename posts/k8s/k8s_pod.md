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

디자인 패턴
=======

- pod에 다중 컨테이너를 디자인 하는 패턴에는 3개가 있다: 사이드카, 앰배서더, 어댑터

Sidecar pattern
-------

- 메인 컨테이너를 보조하는 서브 컨테이너를 포함하는 패턴
- ex) 동적으로 설정을 변경하는 컨테이너, 어플리케이션의 로그 파일을 오브젝트 스토리지로 전송하는 컨테이너

Ambassaddor pattern
------------

- 메인 컨테이너가 외부 시스템과 통신할 때, 통신을 대신 해주는 컨테이너를 포함하는 패턴
- ex) DB에 접속하는 컨테이너

Adaptor pattern
-------

- 서로 다른 데이터 형식을 변환해주는 컨테이너를 포함하는 패턴
- ex) 프로메테우스 등의 모니터링에서 사용하는 형식으로 변환해주는 컨테이너

- - -

init container
==============

- 다른 container가 실행되기 전에 pod를 초기화하는 역할
- pod 설정에 필요한 일을 수행해서 다른 container가 pod 설정과 관련된 코드를 갖지 않아도 되게 하는 역할도 한다
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

- *command*는 도커의 *ENTRYPOINT*를 대체하고, *args*는 *CMD*를 대체한다.

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

생성
---

- Pod를 생성하기 위해서는 아래와 같은 manifest가 필요하다. 아래의 예시는 nginx를 이용하는 container이다

```yaml
#sample-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
    - name: redis-container
      image: redis:latest
```

- 위의 메니페스트를 이용하여 아래와 같이 pod을 만들 수 있다. container가 2개인 것을 볼 수 있다

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG apply -f sample-pod.yaml
pod/sample-pod created
$ kubectl --kubeconfig=$KUBE_CONFIG get pods
NAME                                    READY   STATUS    RESTARTS   AGE
sample-pod                              2/2     Running   0          10s
```

- - -

Health check
=============

- k8s에서는 각 **container**(pod이 아니다)의 상태를 주기적으로 확인하여 문제가 있는 pod을 재시작하거나 서비스에서 제외
- livenessProbe : 컨테이너가 살아있는지 확인. 상태확인이 실패하면 kubelet은 컨테이너를 재시작.
- readinessProbe : 컨테이너가 실행된 후 실제로 서비스 요청에 응답을 할 수 있는지 확인. 실패하면 endpoint controller가 해당 pod가 연결된 모든 service에서 이 pod의 endpoint 정보를 제거(서비스에서 제외된다고 보면된다). pod생성 후 바로 서비스 하는 것이 아닌 별도의 probe를 통해 서비스 전 확인을 한다
- startup probe: pod가 첫번째 시작이 완료되었는지를 확인하고 실패시 다른 probe를 실행하지 않게 한다

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

- healthcheck의 간격을 조절해야할 때가 있다.
  - initialDelaySeconds: 첫 healthcheck까지의 지연. 이 값 대신 되도록 startup probe를 사용하자
  - periodSeconds: healthcheck 간격
  - timeoutSeconds: 타임아웃까지의 시간
  - successThreshold: 성공이라고 판단하기까지의 체크 횟수. livenessProbe와 startup probe는 1 이어야함.
  - failureThreshold: 실패라고 판단하기까지의 체크 횟수


- ReadinessGate: 시스템 전체로 봤을때 pod가 ready인지 확인하는 기능. ReadinessGate를 통과할 때까지 Service 대상에 포함되지 않는다. ReadinessGate가 설정된 deployment를 rolling update 할 때는 ReadinessGate가 Ready가 되지 않으면 다음 pod 변경을 진행하지 않는다
- spec.publishNotReadyAddresses: readinessProbe가 실패하더라도 service를 연결하게 하려면 true로 설정. pod가 ready가 되지 않더라도 service에 이름 해석을 하게 만들 수 있다

- - -

restartPolicy
=======

- 어떠한 이유(정지, health check 실패 등)으로 pod을 재실행할때 어떤 기준으로 재생성할지를 정하는 기준
- spec.restartPolicy:
  - Always: pod가 정지하면 pod를 항상 재시동. Job은 pod 하나당 1회만 실행하므로 Always 불가능
  - OnFailure: 종료 코드 0 이외에는 pod 재기동
  - Never: 파드 재가동 안함

- - -

postStart, preStop
==========

- container의 기동 후와 정지 직전에 실행하는 명령어. 둘 다 exec, httpGet 방식 가능.
- postStart: 컨테이너 기동 후에 시행되는 명령어. entrypoint와 비동기로 거의 같은 시점에 시행됨.
- preStop: 종료 요청이 온 후에 실행
- 두 명령어 모두 최소 한 번 시행되지만, 단 1번 시행된다는 보장은 없다

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pp
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    command: []
    lifecycle:
      postStart:
        exec:
          command: []
      preStop:
        exec:
          command: []
```

- pod에 삭제 요청이 오면 preStop -> SIGTERM 처리가 이루어지며, preStop처리가 시작되는 동시에 service에서 제외되는 처리가 수행된다. 이 제외처리가 SIGTERM 처리까지 이루어지지 않으면 에러가 발생할 수 있으므로 SIGTERM 처리까지 대기 시간(spec.terminationGracePeriodSeconds, default 30초)을 설정하는 것이 좋다.
- SIGTERM 처리가 terminationGracePeriodSeconds 시간 안에 이루어지지 않으면 SIGKILL 신호가 전달되어 강제적으로 종료된다.


- - -

참고자료
======

- <https://arisu1000.tistory.com/27829?category=787056>
