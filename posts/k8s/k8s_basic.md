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


kubectl
========

- kubectl은 k8s master node와 통신시 kubeconfig에 쓰여 있는 정보를 사용하여 통신한다.

```yaml
#kubeconfig 예시
apiVersion: v1
kind: Config
preferences: {}
clusters:
  - name: target-cluster
    cluster:
      server: https://localhost:1234
users:
  - name: my-name
    user:
      client-certificate-data: owefn1e109e
      client-key-data: wefonowf
contexts:
  - name: my-context
    context:
      clusters: target-cluster
      namespace: default
      user: my-name
current-context: my-context
```

- clusters: 접속 대상의 클러스터 정보
- users: 접속하려는 유저의 인증 정보
- contexts: cluster, user, namespace등을 지정

- 위의 kubeconfig는 아래의 명령어로 설정 가능하다

```bash
kubectl config set-cluster target-cluster --server=htts://localhost:1234
kubectl config set-credentials my-name --client-certificate=./sample.crt --client-key=./sample.key --embed-certs=true
kubectl config set-context my-context --cluster=target-cluster --user=my-name --namespace=default
```

- kubectl은 resource등을 생성/관리/삭제 할때도 사용된다

```bash
#생성
kubectl create -f sample-pod.yml #존재하면 ERROR
kubectl apply -f sample-pod.yml #존재하면 변경사항 반영하여 기존의 pod 대체
#확인
kubectl get pods -n namespace
#삭제
kubectl delete pod -f sample-pod.yml
kubectl delete pod pod-name
kubectl delete pod pod-name --wait #동기삭제
#pod 재가동
kubectl rollout restart deployment sample-deployment # deployment의 pod 전부 재가동
```

- apply vs create: 생성 시 사용 가능한 두 가지 방법. 보통은 apply을 사용해야 한다.

- pod 생성시 임의의 이름이 아닌 특정 단어가 prefix로 붙은 이름을 만들고 싶으면 **generateName**을 이용한다

```yaml
# 이 manifest 를 사용하면 "sample-generatename-난수" 의 이름으로 생성된다
apiVersion: v1
kind: Pod
metadata:
  generateName: sample-generatename-
spec:
  containers:
    - name: nginx-container
      image: nginx
```

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

- k8s의 논리적 분리 단위. 가상의 쿠버네티스 클러스터 분리 기능 제공
- Pod, Service는 namespace별로 생성 관리 가능
- 하나의 클러스터에 여러 환경(개발,운영 등)을 관리 가능하게 함
- namespace별로 사용자의 권한을 관리 가능
- 논리적 분리단위이기 때문에 다른 namespace의 pod과 통신은 가능
- 하나의 쿠버네티스 클러스터를 여러 팀에서 사용하거나 dev/stage/real 환경으로 구분하는 경우 사용
- 기본 네임스페이스: 위 3개의 경우 클러스터 관리자가 사용하는 namespace
  - kube-system: 쿠버네티스 클러스터 구성 요소와 애드온이 배포되는 네임스페이스
  - kube-public: 모든 사용자가 사용할 수 있는 configMap 등이 있는 네임스페이스
  - kube-node-lease: node의 heartbeat 정보가 저장되는 네임스페이스
  - default
- namespace 분리는 MSA 개발 팀마다 분리하고 클러스터 별로 나눠야 함
  - dev/stg/real 환경 별로가 아닌 클러스터 별로 나누는 이용은 manifest 재사용성을 높이고 service 이름 해석 시 "service.prd-ns1.svc.cluster.local"과 같은 해석을 방지하기 위함이다

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

k8s 아키텍처
=======

![archi]({{site.baseurl}}/post_img/{{page.name}}/components-of-kubernetes.svg)
[사진 출처](https://kubernetes.io/ko/docs/concepts/overview/components/#%EC%BB%A8%ED%8A%B8%EB%A1%A4-%ED%94%8C%EB%A0%88%EC%9D%B8-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8)

- k8s 아키텍처는 크게 2가지로 나뉜다
  - control plane: k8s cluster을 제어하는 역할. 마스터 노드에 존재하며 etcd, api server, scheduler, control manager, kube-dns 등을 포함
  - worker: pod를 배포하는 node들. kubelet, kube-proxy등을 포함

- - -

control plane
=========

- control plane의 구성요소의 상태는 아래의 방법으로 체크 가능하다

```bash
kubectl --kubeconfig $KUBE_CONFIG get componentstatuses

NAME                 STATUS      MESSAGE                                                                                     ERROR
controller-manager   Unhealthy   Get http://127.0.0.1:10252/healthz: dial tcp 127.0.0.1:10252: connect: connection refused
scheduler            Unhealthy   Get http://127.0.0.1:10251/healthz: dial tcp 127.0.0.1:10251: connect: connection refused
etcd-1               Healthy     {"health":"true"}
etcd-0               Healthy     {"health":"true"}
etcd-2               Healthy     {"health":"true"}
```

etcd
-----

- 분산 key-value store.
- k8s에 등록된 모든 정보가 저장되는 곳.
- api-server를 통해서만 통신 가능.
- optimistic lock 및 유효성 체크 제공: k8s의 리소스를 업데이트 할 때 api server에 metadata.resourceVersion을 제공하고 이를 통해서 apiserver는 lock을 수행한다. api server가 아닌 다른 요소들이 etcd에 접근 할 수 있으면 optimistic lock을 지원하지 않는 다른 요소에 의해 lock이 깨질 수 있다.
- 고가용성을 위해 보통 2 개 이상의 etcd 인스턴스를 실행하기 때문에 일관성 유지는 중요하다. RAFT 알고리즘을 사용하여 두 인스턴스는 합의를 한다. 보통은 합의를 위해서 홀수 대로 배포한다

api server
----------

- k8s의 api를 제공하여 클라이언트와 통신하는 서버
- 모든 구성요소는 api-server하고만 통신한다.
- k8s 상태를 조회, 수정하는 REST API 제공
- etcd를 위한 유효성 체크 및 optimistic lock을 제공.
- 클라이언트 post 요청 -> 인증플러그인 -> 인가 플러그인 -> admission controll plugin
  - admission control plugin: 리소스 수정, 삭제, 생성의 경우 리소스 수정, 재정의 등을 수행. ResourceQuota, AlwaysPullImages가 acp의 예시들
- api server가 controller들에게 무엇을 할지를 알려주지 않고, 상태 변경을 통보 받을 수 있는 요청을 보낼 수 있도록 제공해준다. 이를 통해서 상태를 변경한 것을 확인한 controller들이 스스로 작업을 찾아서 수행한다
  - 여러 클라이언트(controller, scheduler, kubectl 등)들이 watch를 하고 있으면 api server에서 그 watch를 확인할 수 있는 stream을 제공. object에 변경 사항이 생기면 stream을 통해서 object의 새로운 버전을 전달.
- k8s에서 동작하는 모든 형태의 요청과 응답(HTTP/HTTPS rest api)를 통신해주는 역할
- 모든 컴포넌트(scheduler, controller manger, kubelet 등)는 API server를 지켜보다가 자신과 관련 작업을 수행.
  - 예를 들어, pod를 등록하는 경우에는 kubectl에서의 pod 등록 요청 -> API server에서 etcd에 정보 저장 -> scheduler가 kube-apiserver에 정보를 받아서 pod 할당이 필요한 노드를 설정하기 위해 resource의 nodeName을 수정하도록 kube-apiserver에 요청 -> kubelet에서 자기 node에 할당되었지만 동작하지 않는 pod이 있으면 가동시킴.
- kubectl등의 작업도 여기서 수행


kube-scheduler
---------

- spec.nodeName이 설정되지 않은 pod를 찾아내서 kube-apiserver에 업데이트 요청을 보낸다.
- node의 상태, 리소스, taint, toleration, affinity 등의 정보를 가지고 node를 선택
- 여러 개의 scheduler를 동작 시키고 schedulerName에서 선택할 수 있다
- 각 리소스(Pod, Service)들을 node에 배분하는 역할
- spec.nodeName이 할당되어 있지 않은 pod를 감지하고, k8s 클러스터의 node 상태, affinity 등의 조건을 고려하여 node를 선택한다. 그 후에 kube-apiserver에 nodeName 수정 요청
- 리더 선출 방식을 사용하여 하나의 리더만이 write 가능

kube-controller-manager
-----------

- 컨트롤러를 실행하는 역할. 컨트롤러들은 controller manager process 내에서 실행된다. 여기서 컨트롤러는 위에서 살펴본 ReplicaSet, ReplicationController와 다르다.
- 각 컨트롤러는 apiserver를 watch하면서 변겅작업을 수행(ex. deployment controller는 deployment의 상태를 모니터링하면서 replicas 관리). 정기적으로 누락된 이벤트가 없는지 확인한다.
- controller manager가 등록을 하면 scheduler에서 스케쥴링을 수행한다
- 각 컨트롤러들은 pod들을 모니터링(통보 받기 및 주기적으로 pull)하면서 조건과 수가 맞는지 확인하여 pod의 생성 및 삭제를 요청한다. pod의 삭제나 생성이 필요하면 controller manager내의 controller or manager가 pod 생성 요청을 apiserver에 전달한다. 그 후에는 scheduler가 node를 할당하고, kubelet이 pod을 띄운다.

### 1) replication manager

- ReplicationController resource를 다루는 컨트롤러
- ReplicationController가 stream을 통해서 작동 중인 pod의 수를 전달 받아 replicas와 비교 -> 생성 혹은 삭제가 필요하면 api-server에 pod의 생성(manifest를 생성해서) 혹은 삭제 정보를 전달 -> scheduler와 kubelet을 통해서 실행 작업이 수행
- Replicaset controller, DaemonSet controller, job controller도 비슷한 방식으로 동작

### 2) Deployment controller

- deployment resource가 수정될때마다 정해진 방식(카나리, blue-green등)에 따라 새로운 version으로 rollout한다.
- 새로운 버전의 replicaSet resource를 apiserver에 제출한다. replicaSet controller가 resource를 바탕으로 apiserver에 pod 생성 요청.

### 3) StatefulSet controller

- replication manger와 비슷하게 동작하지만, pod뿐만 아니라 PVC도 관리한다.

### 4) node controller

- k8s 클러스터 내부의 node를 기술하는 node resource를 관리.
- 실제 node와 node object 목록을 동기화

### 5) service controller

- LoadBalancer 유형의 서비스가 생성 및 삭제 될 때 인프라에 로드밸런서를 요청 및 삭제한다

### 6) endpoint controller

- service는 pod selector의 정의에 따라 생성된 endpoint 목록을 포함.
- endpoint controller는 label selector와 일치하는 pod의 ip와 port를 endpoint 리스트에 갱신한다(apiserver를 통해서 이루어진다)

### 7) Ingress controller

- reverse proxy를 실행하고 ingress, service, endpoint resource 들을 유지
- 변경사항을 watch하고 reverse proxy server의 설정을 변경한다
- Ingress resource는 service를 가리키지만 ingress controller는 traffic을 직접 service가 아닌 pod로 보낸다

kube-dns
-------

- k8s 내부의 이름 해석, 서비스 디스커버리에 사용되는 내부 DNS 서버.
- apiserver와 연계하고 있어서 서비스가 생성되었을 때나 서비스에 연결된 pod가 변경되면 DNS 설정을 변경

- - -

worker node의 구성요소
=============

- k8s의 모든 worker node에서 동작하는 kubelet, kube-proxy가 있다

kubelet
-------

- node마다 하나씩 존재하는 것. node에 할당되면 node에 대한 node resource를 만들어서 API 서버에 등록
- container runtime과 연계하여 container을 생성, 정지 등을 관리하는 역할
- api-server가 pod 등록 -> scheduler가 node 선택 -> kubelet이 자신의 node에 container 생성.
- Static-pod는 scheduler와 apiserver를 거치지 않고 local directory의 manifest로 node에 배포(control plane의 구성 요소 등)
- 시스템 구성요소인 데몬으로 실행되는 유일한 k8s 구성 요소. 마스터 노드에도 배포된다
- 실행중인 컨테이너를 모니터링 하면서 상태, 리소스 점유율 등을 apiserver에 전달
- livenessProbe을 실행하는 역할도 수행

- pod가 생성되면, 해당 node에 *pod infrastructure container*가 생성되고, 나머지 container들이 이 container 내부에(이 container의 namespace 를 갖는 상태) 생성된다. 같은 pod내의 컨테이너들은 같은 namespace를 갖기 때문에 localhost로 통신 가능하다.

kube-proxy
-------

- ClusterIP, NodePort로 가는 traffic이 pod에 전송될 수 있도록 해준다(서비스의 ip와 port)
- userspace, iptables(기본값), ipvs 모드가 존재
  - userspace: 실제 proxy 서버를 생성하고 traffic을 proxy 서버에서 경로를 변경하는 방법. kernel과 userspace 사이의 이동이 추가되어서 성능이 좋지 않음. 실제로 RR 방식을 사용
  - iptables: iptables을 수정하여 proxy 역할을 수행. kernel영역에서 수행됨. 대규모 클러스터에서는 성능이 좋지 않음. RR 방식을 사용하지 않고 무작위로 보냄
  - ipvs: rr이외의 loadbalancing(least connection 등)을 제공.

- - -

몇 가지 사례
========

Deployment controller
----------

1. 새로운 deployment 생성
2. API 서버가 watch 중인 클라이언트들(deployment controller 포함)에게 정보 전달
3. Deployment controller가 apiserver에 ReplicaSet 생성 통보
4. ReplicaSet Controller가 watch하고 있다가 ReplicaSet 정보 확인
5. ReplicaSet controller가 apiserver에 pod 생성 요청 전달
6. watch 중이던 scheduler가 적절한 node를 pod에 할당하는 요청(pod의 nodeName을 변경)을 apiserver에 전달
7. watch 중이던 kubelet이 자신의 node에 pod가 할당되면 container runtime에 컨테이서 생성 요청
8. container runtime이 container 생성

```bash
$ kubectl --kubeconfig $KUBE_CONFIG get events

LAST SEEN   TYPE      REASON                         OBJECT                                      MESSAGE
63s         Normal    Scheduled                      pod/sample-deployment-5d7cb6947d-9zsxl      Successfully assigned default/sample-deployment-5d7cb6947d-9zsxl to nks-sdfasdfsf-w-mgx
28s         Normal    Pulling                        pod/sample-deployment-5d7cb6947d-9zsxl      Pulling image "nginx:latest"
25s         Normal    Pulled                         pod/sample-deployment-5d7cb6947d-9zsxl      Successfully pulled image "nginx:latest"
25s         Normal    Created                        pod/sample-deployment-5d7cb6947d-9zsxl      Created container container1
63s         Normal    SuccessfulCreate               replicaset/sample-deployment-5d7cb6947d     Created pod: sample-deployment-5d7cb6947d-9zsxl
63s         Normal    ScalingReplicaSet              deployment/sample-deployment                Scaled up replica set sample-deployment-5d7cb6947d to 1
```

- pod가 생성되면 manifest에 정의된 container외에도 **pause container**가 생성된다
  - pause container: pod의 리눅스 네임스페이스를 유지하는 infrastructure container. pod 내의 다른 container들은 이 container의 namespace를 사용. 새로운 컨테이너가 생성되면 pause container의 네임스페이스를 사용

- - -

참고자료
==========
- <https://bcho.tistory.com/1257?category=731548>
- <https://blog.2dal.com/2018/03/28/kubernetes-01-pod/>
- <https://arisu1000.tistory.com/27829?category=787056>
- [쿠버네티스 공식 문서](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)
