---
layout: post
title:  "service"
date:   2019-12-18 01:02:59
name: k8s_service.md
---

Service
=========

- Pod은 IP가 랜덤하게 지정되고 재시작때마다 IP가 변함. -> 고정된 endpoint 사용 어려움
- service가 pod에 대한 접근 방법과, 로드 밸런서 역할을 수행
- label selector를 이용하여 관리하고자 하는 Pod을 지정
- 여러 port를 여러 protocol로 오픈 가능

```yml
apiVersion: v1
kind: Service
metadata:
  name: hello-node-svc
spec:
  selector:
    app: hello-node
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
    - name: https
      port: 443
      protocol: TCP
      targetPort: 8082
  type: LoadBalancer
```

템플릿
----

```yml
kind: Service
apiVersion: v1
metadata:
  name: my-service #서비스 이름
spec:
  type: ClusterIP #서비스 종류(기본값 = ClusterIP
  clusterIP: 10.0.10.10 #클러스터 내부에서 사용되는 ㅑㅖ
  selector:
    app: MyApp #관리할 pod의 label
  ports:
  - protocol: TCP
    port: 80 #내부에서 접근하는데에 사용되는 port
    targetPort: 9376 #실제로 트래픽을 전달받는 port
```
종류
---

#### 1) ClusterIP

- 가장 기본적인 타입, 클러스터 내부에서 사용가능
- 클러스터 내부의 node와 pod에서 이 clusterIP를 이용해서 서비스에 연결된 pod에 접속
- 클러스터 외부에서는 사용 불가

#### 2) LoadBalancer

- AWS같은 cloud서비스를 사용할 때 가능한 옵션.
- pod를 cloud에서 제공해주는 load balancer와 연동. 그 load balancer의 IP를 통해서 외부에서 접근

#### 3) NodePort

- 각 node에 port를 할당하는 방식.
- node자체에 port가 존재하기때문에 외부에서 내부 pod으로 직접 접근할 수 있다

```yml
apiVersion: v1
kind: Service
metadata:
  name: hello-node
spec:
  selector:
    app: hello-node
  type: NodePort
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
      nodePort: 30036 #외부에서 30036 port 로 직접 node에 접근 가능
```
- port : 클러스터 내부에서 사용하는 port. 여기로 전달된 요청은 targetPort로 전달된
- targetPort : 실제로 사용하는 port. 트래픽을 수신하는 port

#### 4) ExternalName

- 외부 서비스를 k8s 내부에서 호출할 때 사용

```yml
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

들어온 요청을 my.database.example.com으로 포워딩 해준다
