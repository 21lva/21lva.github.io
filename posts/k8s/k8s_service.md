---
layout: post
title:  "Service, Ingress"
date:   2019-12-18 01:02:59
name: k8s_service.md
category: kubernetes
---

Service
=========

- Pod은 IP가 랜덤하게 지정되고 재시작때마다 IP가 변함. -> 고정된 endpoint 사용 어려움
- Service가 pod에 대한 접근 방법과, 로드 밸런서(L4) 역할을 수행
- label selector를 이용하여 관리하고자 하는 Pod을 지정
- 여러 port를 여러 protocol로 오픈 가능

- 같은 Pod내에 있는 container는 localhost를 통해서 통신이 가능. But 다른 pod의 컨테이너끼리는 pod의 ip를 통해서 통신이 이루어져야한다

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

- 가장 기본적인 타입, 클러스터 내부에서 사용가능(외부에서는 사용 불가능)
- 클러스터 내부의 node와 pod에서 이 clusterIP를 이용해서 서비스에 연결된 pod에 접속
- ClusterIP로 생성 및 지정되는 IP는 실제 IP가 아닌 virtural IP이다. 이 virtual IP을 이용한 통신은 kube-proxy가 담당한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-clusterip
spec:
  #type을 지정하지 않으면 default로 ClusterIP로 지정된다
  ports:
  - name: http-port
    protocol: TCP
    port: 8080
    targetPort: http #container의 http로 전송
  selector:
    app: sample-app


---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - name: http
          containerPort: 80
```

- 위의 메니페스트로 생성한 pod과 서비스 정보는 아래와 같다

```bash
$
kubectl --kubeconfig=$KUBE_CONFIG get pods -o wide
NAME              READY   STATUS    RESTARTS   AGE     IP             
nginx-68bccfcf5d-prz25   1/1     Running   0          60s   198.18.0.183
nginx-68bccfcf5d-t47k9   1/1     Running   0          60s   198.18.0.195
$ kubectl --kubeconfig=$KUBE_CONFIG get svc
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
sample-clusterip   ClusterIP   198.19.254.233   <none>        8080/TCP   51s
```

- 생성된 service를 확인하면 endpoint로 생성된 pod으로 연결되어 있는 것을 볼 수 있다

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG describe svc/sample-clusterip
Name:              sample-clusterip
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=sample-app
Type:              ClusterIP
IP Families:       <none>
IP:                198.19.254.233
IPs:               <none>
Port:              http-port  8080/TCP
TargetPort:        http/TCP
Endpoints:         198.18.0.183:80,198.18.0.195:80
Session Affinity:  None
Events:            <none>
```

- ClusterIp는 같은 클러스터내에서만 접속이 가능하기 때문에 새로운 pod을 만들어서 통신해야 한다.

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG exec -it sample-pod2 /bin/bash
root@sample-pod2:/# curl -X GET http://198.19.254.233:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

#### 2) NodePort

- 각 node에 port를 할당하는 방식.
- node자체에 port가 존재하기때문에 외부에서 내부 pod으로 직접 접근할 수 있다
- ExternalIP를 사용하면 특정nodeIP:port로 수신한 패킷을 컨테이너로 전송하지만, NodePort를 이용하면 **모든**nodeIP:port로 수신
- NodePort는 ClusterIP를 포함한다

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
      nodePort: 30036 #외부에서 30036 port 로 직접 node에 접근 가능. 숫자를 지정하지 않으면 랜덤으로 생성된다
```
- port : 클러스터 내부에서 사용하는 port. 여기로 전달된 요청은 targetPort로 전달된
- targetPort : 컨테이너의 port
- 수신: nodePort -> port -> targetPort

#### 3) LoadBalancer

- AWS같은 cloud서비스를 사용할 때 가능한 옵션.
- k8s cluster 외부의 로드 밸런서에 외부와 통신 가능한 가상 IP를 할당. **NodePort** 서비스를 생성하고 외부의 로드밸런서에서 이 NodePort 서비스로 로드밸런싱하는 형태.
- 고정 IP를 사용하고자 한다면 IP주소를 지정할 수 있다(GCP, AWS같은 클러스터 공급자가 허용한다면)

```yml
apiVersion: v1
kind: Service
metadata:
  name: sample-lb
spec:
  selector:
    app: sample-app
  type: LoadBalancer
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
      nodePort: 30036
```

- port: clusterIP와 LoadBalancer에서 사용하는 port. 외부 로드 밸런서에 할당되는 가상 IP와 ClusterIP의 가상 IP가 사용하는 port는 같다.
- nodePort: node에서 사용하는 port
- targetPort: Container에 요청이 전달된 포트
- lb의 port -> nodePort -> cluster의 port -> 컨테이너의 targetPort


#### 4) Headless service

- 개별 pod의 ip 주소가 직접 반환되는 서비스
- 위의 ip들을 전부 가상 IP 였지만, 헤드리스 서비스에서는 ip가 제공되지 않고 DNS를 통해서 엔드포인트를 제공한다
  - DNS 캐싱을 조심해야함
- StatefulSet은 pod 이름을 통해서 ip 주소를 확인할 수 있다.


- 헤드리스 서비스를 만들기 위해서는 **spec.type=ClusterIP, spec.clusterIP=None** 으로 지정해야한다

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-clusterip
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: http-port
    protocol: TCP
    port: 8080
    targetPort: http
  selector:
    app: sample-app
```

#### 5) ExternalName

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

- pod에서는 my.database.example.com으로 직접 요청을 보내는 것이 아니라 my-service.default.svc.cluster.local로 요청을 보낸다. 서비스는 들어온 요청을 my.database.example.com으로 포워딩 해준다. app에 외부 도메인이 아닌 서비스 주소를 endpoint로 설정하게 해서 외부 서비스의 주소가 변경되는 등의 변화에 대응하기 쉬워진다.
- FQDN을 지정해 놓고 DNS 해석을 통해서 CNAME나 CluterIP을 반환해서 내부서비스와 외부서비스 사이의 전환을 용이하게 해준다

서비스 연결
--------

- service는 내부 DNS 서버에서 DNS 정보를 가져온다. Pod는 FQDN을 통해서 접근할 수 있다
- **service이름.네임스페이스.svc.cluster.local**. 같은 네임스페이스에 있는 경우에는 **svc.cluster.local**을 생략할 수 있다.

- - -

kube-proxy
============

- kube-proxy는 모든 worker node에서 실행되면서 서비스의 ip, port로 들어온 패킷을 pod로 연결시킨다.
- pod간의 loadbalancing 기능도 제공

userspace를 사용하는 경우
-------

- kube-proxy는 초기에 usernspace을 사용하여 동작하였다
- iptables를 수정해서 프록시 서버로 전달하게 하고 이를 다시 서버로 전달하는 방식이었다.
- 실제 프록시 서버이기 때문에 kube-proxy라는 이름이 명명된 것
- userspace와 kernel space을 왔다갔다 해야해서 성능이 떨어짐

iptables proxy 모드
----------

- 현재 주로 사용되는 구현
- iptables의 규칙만을 수정해서 프록시 서버를 거치지 않고 패킷을 무작위로 선택된 pod로 전달하게 한다
- userspace 모드에서는 실제 프록시 서버로 인해서 RR을 수행하지만, iptables 모드에서는 무작위로 선택된다.

- - -

Pod끼리의 네트워킹
=====

- Pod끼리 혹은 node끼리 그리고 pod와 node사이에서의 통신은 NAT 없이 이루어진다
- 외부로의 패킷은 호스트 worker node의 ip로 설정된다
- 한 pod내의 container들은 같은 veth에 연결된다(localhost로 통신 가능)

Node내에서의 통신
-----------

- Pod가 생성될 때 veth 한 쌍이 생성된다(하나는 node의 namespace에, 하나는 container의 네임스페이스에 eth0이라는 이름으로 위치)
- 이 veth은 node의 bridge에 연결됨. 이 bridge를 통해서 같은 node내의 pod끼리의 통신이 가능함

Node끼리의 통신
------------

- 한 클러스터 내의 pod의 ip는 유일한다. 그러므로 충돌을 방지하기 위하여 각 node의 bridge들은 서로 겹치지 않는 주소 범위를 사용해야 한다.
- 각 노드의 bridge들은 다양한 네트워크(오버레이, 언더레이, L3 라우팅등을 통해 구현)에 연결되어 있다.
  - bridge는 eth0와 연결되어 있고 이 eth0의 다른 한쪽이 네트워크와 연결되어 있다

- - -

Service의 통신
=========

- 서비스에 생성된 ip는 가상 ip이기때문에 출발지나 도착지 ip주소로 표시되지 않는다.
- service는 ip:port의 쌍으로 이루어져있기 때문에 ping을 보낼 수 없다

- Service가 생성되면 가상 IP가 할당 -> 모든 node의 kube-proxy에 새로운 Service 생성을 알림 -> kube-proxy에서 iptables을 수정해서 패킷이 pod로 리다이렉트 하도록 변경

- serviceIp:port로 패킷을 전송하면 해당 node에서 iptables를 확인해서 임의로 ip와 port를 선택해서 목적지IP와 포트를 수정한다. 이때 수정작업은 netfilter에 의해 이루어진다. 수정된 패킷은 node의 eth0을 거쳐서 다른 node으로 전달된다

- 외부에서 요청이 들어오면 LB에 의해서 NodePort로 연결된다 -> NodePort로 들어온 traffic은 kube-proxy에 의해서 serviceIp:port로 변경된다 -> serviceIp:port는 다시 node내 적절한 pod의 ip:port로 변경되고 전달된다

- - -

Ingress
=======

- 클러스터 외부에서 내부로 접근하는 요청들을 처리하는 규칙. HTTPs 기반의 L7 로드 밸런싱 기능 제공
- URL 사용하게 해줌, traffic 로드 밸런싱, ssl인증서 처리, 도메인 기반으로 가상 호스팅 제공
- Ingress controller : ingress(resource) 규칙을 기반으로 동작하게 하는 controller.
- k8s에서는 리소스와 컨트롤러가 있다. 클러스터에 리소스가 등록되면 컨트롤러가 해당 리소스 내용을 실제로 클러스터에 반영한다. 즉, 컨트롤러가 실제로 lifecycle을 관리하는 것
- URL 기반의 라우팅은 API 게이트웨이가 아닌 Ingress로 하는 것이 좋다

- aws같은 cloud 서비스를 이용하면 자체 로드밸런서 서비스들과 연동해서 쉽게 사용 가능
  - 클라이언트 -> L7 로드 밸런서 -> NodePort -> pod
- 자체적으로 운영할 경우에는 ingress와 ingress controller를 연동해야 함
  - 클라이언트 -> L4 로드 밸런서(기존의 LoadBalancer타입) -> nginx pod(controller. 말은 controller이지만 실제 처리도 함) -> target pod
- ingress controller는 모든 ingress resource를 볼 수 있으므로 충돌이 발생할 수 있다. 이럴 때는 *kubernetes.io/ingress.class* 값을 지정하여 분리하면 된다.


```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: a.b.com
    http:
      paths:
      - path: /svc1
        backend:
          serviceName: s1
          servicePort: 80
      - path: /svc2
        backend:
          serviceName: s2
          servicePort: 80
  - host: c.d.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
```

위는 ingress 설정이다
- **a.b.com** 에 요청이 들오고 **a.b.com/svc1** 으로 요청이 들어오면 s1 서비스로 라우팅해주고 **a.b.com/svc2** 에 요청이 들어오면 s2 서비스로 라우팅해준다.
- **c.d.com** 에 요청이 들어오면 s2 서비스로 라우팅 할 수 있다

- - -

Nginx Ingress controller in Minikube
===========================


- [minikube용 ingress](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)를 참고하였다.
- minikube가 아닌 다른 driver을 위한 ingress controller 설정법은 [여기](https://kubernetes.github.io/ingress-nginx/deploy/)를 참고하자

먼저 minikube의 addon을 실행하자

```
minikube addons enable ingress
```

```
kubectl get pods --all-namespaces
```

를 통해서 **nginx-ingress-controller** 가 존재하는 것을 확인할 수 있다

![getIngress]({{site.baseurl}}/post_img/{{page.name}}/getIngress.png)

Deployment를 서비스해보자. Ingress로 관리하고자 하는 서비스는 꼭 **NodePort** 타입으로 만들어야한다

```
kubectl run web --image=gcr.io/google-samples/hello-app:1.0 --port=8080
kubectl expose deploy web --target-port=8080 --type=NodePort
```

아래의 명령어로 만들어진 서비스의 ip를 알 수 있다

```
minikube service web --url
```

ingress 설정 파일을 만들어보자

```yaml
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - host: hello-world.info
    http:
      paths:
      - path: /
        backend:
          serviceName: web
          servicePort: 8080
```

다음의 명령어로 ingress를 만들자

```
kubectl apply -f example-ingress.yaml
```

kubectl get ingree를 통해서 주소값(address)을 얻자. 로컬에서 실행하는 경우에는 **minikube ip** 를 실행해서 외부 주소를 사용하자(위의 주소는 내부 ip)

/etc/hosts 파일을 열고(리눅스 기준) 아래의 내용을 적는다

```
<ip 주소> hello-world.info
```

웹브라우저를 열고 **hello-world** 를 입력하면 결과 화면을 볼 수 있다

- - -

Nginx Ingress in NKS
=======

- NKS에서 nginx ingress를 사용해보자
- 먼저 ingress controller가 배포되어있는지 확인한다

```bash
kubectl --kubeconfig=$KUBE_CONFIG get pods --all-namespaces
NAMESPACE       NAME                                        READY   STATUS             RESTARTS   AGE
ingress-nginx   ingress-nginx-controller-66dc9984d8-nclpz   1/1     Running            0          8d
```

- 아래는 ingress 리소스, 서비스, pod을 등록하기 위한 manifest이다. 이 때, Service의 type은 **NodePort**로 지정한다.

```yaml
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - host: hello-world.info
    http:
      paths:
      - path: /app1
        backend:
          serviceName: svc-1
          servicePort: 8888
      - path: /app2
        backend:
          serviceName: svc-2
          servicePort: 8888

---

apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  type: NodePort
  ports:
  - name: "http"
    protocol: TCP
    port: 8888
    targetPort: 80
  selector:
    ingress-app: app-1

---

apiVersion: v1
kind: Pod
metadata:
  name: app-1
  labels:
    ingress-app: app-1
spec:
  containers:
  - name: nginx-container
    image: nginx:latest

---

apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  type: NodePort
  ports:
  - name: "http"
    protocol: TCP
    port: 8888
    targetPort: 80
  selector:
    ingress-app: app-2

---

apiVersion: v1
kind: Pod
metadata:
  name: app-2
  labels:
    ingress-app: app-2
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
```

- 배포하고 확인해보자.

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG apply -f ingress.yaml
ingress.networking.k8s.io/example-ingress created
$ kubectl --kubeconfig=$KUBE_CONFIG get ingress
NAME              CLASS    HOSTS              ADDRESS                                                                  PORTS   AGE
example-ingress   <none>   hello-world.info   ingress-ngi-ingress-ngin-fd98c-8233885-c3a67cad4003.kr.lb.naverncp.com   80      6s
```


참고자료
=======
- [kubernetes in action](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791161754048&orderClick=JAK)
- [네트워크 관련 포스트](https://coffeewhale.com/k8s/network/2019/04/19/k8s-network-01/)
