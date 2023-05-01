---
layout: post
title:  "K8s - Manifest"
date:   2021-09-26 01:02:59
author: Inhyuk
category: k8s
tags: k8s
cover:  "/assets/instacode.png"
name: k8s_manifest.md
---

annotation vs label
===========

- 두 경우 다 metadata를 제공한다

annotation
-------

- **metadata.annotation**로 설정
- 시스템 구성 요소가 사용하는 정보, 모든 환경에서 사용할 설정 등을 제공
- key-value로 값을 저장한다고 생각하면 된다.

label
------

- **metadata.labels**로 설정
- 리소스를 구분하기 위한 정보

```yml
apiVersion: v1
kind: Pod
metadata:
  name: sample
  labels:
    label1: val1
    label2: val2
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
```

- 아래의 방법으로 특정 label을 가지는 pod만 확인 가능

```bash
$ kubectl get pods -l label1=val1
```

- 이 label들을 이용하여 replicaset이 pod의 수를 관리하고, Service에서 pod의 routing을 관리하고, loadbalancer가 목적지 pod를 결정한다

- - -

Helm
======

- k8s package(chart) 관리자.
- helm에서는 chart 단위로 시스템을 packaging한다. 새로운 차트를 만들면서 알아보자

- 아래는 간단한 k8s 클러스터 생성을 위한 go web server이다
- 아래의 내용은 **Naver cloud platform** 에서 **Container registry**와 **Kubernetes service**를 이용하였다.

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

const (
	port = 8080
)

func main(){
	http.HandleFunc("/hello", sayHello)

	if err := http.ListenAndServe(fmt.Sprintf(":%d", port),nil); err != nil {
		log.Fatalln(err)
	}
}

func sayHello(response http.ResponseWriter, req *http.Request){
	if _, err := response.Write([]byte("Hello")); err != nil {
		log.Fatalln(err)
	}
}
```

- 여기에 chart를 설치해보자

```bash
user@AL01749687  ~/GolandProjects/ginExample  helm create helm-chart
Creating helm-chart
user@AL01749687  ~/GolandProjects/ginExample  ll
total 16
-rw-r--r--  1 user  staff    39B  9 26 15:05 go.mod
drwxr-xr-x  7 user  staff   224B  9 26 15:23 helm-chart
-rw-r--r--  1 user  staff   374B  9 26 15:22 main.go
user@AL01749687  ~/GolandProjects/ginExample  tree .
.
├── go.mod
├── helm-chart
│   ├── Chart.yaml
│   ├── charts
│   ├── templates
│   │   ├── NOTES.txt
│   │   ├── _helpers.tpl
│   │   ├── deployment.yaml
│   │   ├── hpa.yaml
│   │   ├── ingress.yaml
│   │   ├── service.yaml
│   │   ├── serviceaccount.yaml
│   │   └── tests
│   │       └── test-connection.yaml
│   └── values.yaml
└── main.go

4 directories, 12 files
```

- 기본적으로 새로 만든 chart에는 아래의 정보를 포함한다
  - **Chart.yaml**: 헬름 차트에 대한 메타데이터
  - **values.yaml**: 사용자가 다른 메니페스트에서 사용할 값을 정의해 놓은 곳. 리소스, 이미지, 포트 등을 지정할 수 있다
  - **templates/~~.yml**: Service, deployment 등을 정의해놓은 메니페스트
  - **templates/tests/~~.yml**: 차트가 제대로 동작하는지 테스트하는 manifest
  - **templates/NOTES.txt**: helm install시 출력되는 메시지
  - **requirements.yaml**: 의존하는 차트에 대한 정보
  - **templates/_helpers.tpl**: 릴리스 이름으로 변수를 정의하는 곳
  - **charts/**: requirements.yaml에서 사용할 다른 차트를 포함하는 디렉토리

- docker image를 생성하기 위한 **Dockefile**을 만든다

```Dockerfile
FROM golang:1.16

WORKDIR /go/src/app
COPY . .
RUN go build main.go

CMD ["./main"]
```

- docker 이미지를 생성하고 private registry에 올린다(생략)
- private image를 k8s에서 사용하려면 secret을 생성하고 그것을 **imagePullSecrets**에 넣어야한다

```bash
$ kubectl create secret docker-registry regcred --docker-server=<registry-end-point> --docker-username=<access-key-id> --docker-password=<secret-key> --docker-email=<your-email>
```

```yaml
# values.yaml

# 위 생략
imagePullSecrets:
  - name: regcred
nameOverride: ""
fullnameOverride: ""
# 아래 생략
```

- 아래의 내용처럼 helm chart를 조금 수정한다

```yaml
#values.yaml

#위 생략
image:
  repository: o49tnvrs.kr.private-ncr.ntruss.com
  name: go-server
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "1.0"

#중간 생략

service:
  type: LoadBalancer
  port: 80
  targetPort: 8080

#아래 생략
```

```yaml
#deployment.yaml

#위 생략
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}/{{ .Values.image.name }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
#아래 생략
```

- NCP에서 제공하는 LoadBalancer를 사용하기 위해서는 **service.yaml**을 아래처럼 수정한다. [관련 내용](https://guide.ncloud-docs.com/docs/vnks-nks-1-8)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "helm-chart.fullname" . }}
  annotations:
    service.beta.kubernetes.io/ncloud-load-balancer-layer-type: "nlb"
    service.beta.kubernetes.io/ncloud-load-balancer-internal: "false"
    service.beta.kubernetes.io/ncloud-load-balancer-size: "SMALL"
  labels:
    {{- include "helm-chart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
      name: http
  selector:
    {{- include "helm-chart.selectorLabels" . | nindent 4 }}
```

- 이를 helm 으로 배포한다

```bash
$ helm --kubeconfig=$KUBE_CONFIG install go-server ./helm-chart
```

- 제대로 배포 되었는지 확인해보자

```bash
 ~/GolandProjects/ginExample  kubectl --kubeconfig=$KUBE_CONFIG get svc
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)        AGE
go-server-helm-chart   LoadBalancer   198.19.231.172   default-go-server-helm-c-3cefb-8233976-8142443f972f.kr.lb.naverncp.com   80:31070/TCP   3h22m
 ~/GolandProjects/ginExample  kubectl --kubeconfig=$KUBE_CONFIG get pods
NAME                                    READY   STATUS    RESTARTS   AGE
go-server-helm-chart-6bb6d95b4c-n6jx7   1/1     Running   0          9m54s
 ~/GolandProjects/ginExample  kubectl --kubeconfig=$KUBE_CONFIG get deployment
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
go-server-helm-chart   1/1     1            1           3h22m
```

- 요청을 보내보자

```bash
 curl -X GET http://default-go-server-helm-c-3cefb-8233976-8142443f972f.kr.lb.naverncp.com/hello
Hello. This is a GO Server.%
```

- *helm template*: 클러스터에 어플리케이션을 생성하지는 않고, helm chart들을 바탕으로 k8s의 manifest들만 생성. 그 대상들을 *heml apply*를 통해서 클러스터에 적용 가능.

helm architecture
---------------

- helm client는 chart와 values의 조합을 release로 관리.

- - -

livenessProbe vs readinessProbe
=========

- kubelet이 컨테이의 상태를 확인하는 방법

livenessProbe
-----

- 컨테이너가 동작 중인지를 확인. 실패하면 해당 Pod를 죽이고 새로운 Pod를 재시작한다
- **initialDelaySecond**만큼 대기 후에 **periodSecond**에 정해지 주기에 따라 상태를 체크한다
- HTTP, TCP, 명령어 수행의 3가지 방법이 존재

readinessProbe
-------

- container가 일시적으로 이용 불가능한 것을 확인하는 방법
- 실패시 해당 pod에 요청을 보내지 않도록 서비스에서 ip를 제외한다
