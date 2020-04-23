---
layout: post
title:  "Ingress"
date:   2019-12-18 01:02:59
name: k8s_ingress.md
category: kubernetes
---


Ingress
=======

- 클러스터 외부에서 내부로 접근하는 요청들을 처리하는 규칙. HTTPs 기반의 L7 로드 밸런싱 기능 제공
- URL 사용하게 해줌, traffic 로드 밸런싱, ssl인증서 처리, 도메인 기반으로 가상 호스팅 제공
- Ingress controller : ingress 규칙을 기반으로 동작하게 하는 controller
- URL 기반의 라우팅은 API 게이트웨이가 아닌 Ingress로 하는 것이 좋다

- aws같은 cloud 서비스를 이용하면 자체 로드밸런서 서비스들과 연동해서 쉽게 사용 가능
- 자체적으로 운영할 경우에는 ingress와 ingress controller를 연동해야 함


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

Ingress controller with nginx
===========================


- [여기](https://arisu1000.tistory.com/27840?category=787056)에 나온 것은 minikube와 맞지 않아서 [minikube용 ingress](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)를 참고하였다.
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

Ingress with TLS
================

추가 예정
