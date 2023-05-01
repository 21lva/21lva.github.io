---
layout: post
title:  "Autoscaler"
date:   2019-12-18 01:02:59
name: k8s_autoscaler.md
category: kubernetes
---

Autoscaler
==========


- node를 추가하는 autoscaler
- cluster Autoscaler: k8s 클러스터 자체의 오토 스케일링.
- cluster Autoscaler는 클러스터 전체나 각 node의 부하가 높아졌을 때 하는 것이 아니라, pod가 pending되는 시점에 requests와 limits를 바탕으로 동작한다. 적절하지 않는 requests와 limits는 실제 사용량이 적은 상태에서 이루어지는 scale out과 실제 사용량이 높은 상태에서의 scale in을 야기할 수 있다
- 기본적을 requests를 기준으로 동작. requests를 초과하여 할당하는 경우에 신규 node를 추가한다. 만약에 limits와 requests의 차이가 커서 실제 사용량과 다르게 requests가 작으면 scale out이 이루어지지 않아서 문제가 생길 수도 있다. 그러므로 실제로는 requests와 limits를 차이가 작게 + requests를 작게 설정한 후에 성능 테스트를 통해서 점점 증가시켜야한다. 메모리의 경우에는 OOMKilled가 일어나지 않도록 **적당히 작은 requests**를 설정해야 한다

- - -

Pod의 Autoscaler
===============

HorizontalPodAutoscaler
-----------

- Deployment, Replicaset, Replication controller의 replica 수를 cpu 부하 등에 따라 자동으로 scaling는 리소스
- pod에 resource requests가 설정되어 있지 않으면 동작하지 않는다
- 주기(30초)적으로 필요한 레플리카 수를 생각해서 스케일링 여부를 확인. 필요한 replicas 수 = (현재 cpu 사용률 총합/기대되는 평균 효율) 보다 큰 값.
  - CPU 사용률은 metrics-serverdㅔ서 가져온 각 pod의 1분간 평균 값 사용.
  - scale in은 5분에 한 번, scale out은 3분에 최대 1번 -> scale in보다 scale out이 중요하기 때문
- pod와 node의 metric은 node안에서 실행되는 kubelet의 cAdvisor에서 수집된다. 이 클러스터의 node하나에서 pod로 실행되고 있는 지표들은 heapster에 모이고 hpa는 heapster에 REST로 질의를 보내서 metric들을 받아온다.

- 아래는 hpa 예제를 위한 manifest이다

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: sample-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sample-hpa-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 20

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-hpa-deployment
spec:
  replicas: 1 #초기값
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: container1
        image: busybox:latest
        command: ["dd","if=/dev/zero","of=/dev/null"]
        resources:
          requests:
            memory: "1024Mi"
            cpu: "500m"
```

- 이를 배포하고 확인하면 pod의 수가 변한 것을 볼 수 있다

```bash
kubectl --kubeconfig=$KUBE_CONFIG get all
NAME                                        READY   STATUS    RESTARTS   AGE
pod/sample-hpa-deployment-8645b7497-bv2nm   0/1     Pending   0          70s
pod/sample-hpa-deployment-8645b7497-cddrs   1/1     Running   0          2m33s
pod/sample-hpa-deployment-8645b7497-jr7qz   0/1     Pending   0          85s
pod/sample-hpa-deployment-8645b7497-qnkh6   1/1     Running   0          85s
pod/sample-hpa-deployment-8645b7497-tzp68   0/1     Pending   0          85s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sample-hpa-deployment   2/5     5            2           10m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/sample-hpa-deployment-8645b7497   5         5         2       2m33s

NAME                                             REFERENCE                          TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/sample-hpa   Deployment/sample-hpa-deployment   193%/20%   1         5         5          10m
```

- hpa는 deployment의 replica 수를 변경해서 autoscaling을 진행한다

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG describe deployment.apps/sample-hpa-deployment
Name:                   sample-hpa-deployment
Namespace:              default
CreationTimestamp:      Sat, 02 Oct 2021 11:04:34 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=sample-app
Replicas:               5 desired | 5 updated | 5 total | 2 available | 3 unavailable
```

- (hpa는 다양한 조건과 동작을 설정할 수 있다)[https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale/]
  - spec.behavior.scaleDown: scale in 하는 조건, 가능한 레플리카 수의 백분율 등
  - spec.behavior.scaleUp: scale out 하는 조건, 가능한 레플리카 수의 백분율 등

VeriticalPodAutoscaler
-------------------

- hpa에서는 requests를 설정하지 않으면 scailing되지 않기 때문에 requests를 필수로 설정해야 하지만 이 값은 실제 서비스 환경에서 배포해서 성능을 테스트 하지 않으면 적절한 값을 찾기 쉽지 않다. 적절하지 않는 requests는 리소스 부족 또는 리소스 과잉으로 이어질 수 있다.
- vpa에서는 컨테이너에 할당하는 cpu, memory을 자동으로 scale up/down 해준다

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: sample-hpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sample-vpa-deployment
  updatePolicy:
    updateMode: Auto # Requests 업데이트 하면 pod 재생성
  resourcePolicy:
    containerPolicies:
    - containerName: container1 #제외 컨테이너
      mode: "Off"
    - containerName: "*" #특정되지 않은 모든 컨테이너
      mode: Auto
      minAllowed:
        cpu: 300m
      maxAllowed:
        cpu: 500m
      controlledResources: ["cpu"]

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-vpa-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: container1
        image: busybox:latest
        command: ["dd","if=/dev/zero","of=/dev/null"]
        resources:
          requests:
            memory: "1024Mi"
            cpu: "100m"
      - name: container2
        image: busybox:latest
        command: ["dd","if=/dev/zero","of=/dev/null"]
        resources:
          requests:
            memory: "1024Mi"
            cpu: "100m"
```

- updateMode: requests를 변경하기 위해 pod를 재생성하는 정책
  - Off: requests 추천값을 계산하지만 실제 변경은 하지 않음
  - Initial: pod를 재생성하는 시점에만 변경.
  - Recreate: 추천값이 변경될 때 pod 재생성하고 requests값 변경
  - InPlace: 추천값입 변경될 때 기존 pod의 requests값 변경
  - Auto: 추천값이 변경될 때 Recreate or InPlace로 알아서

- 위 기능은 아직 베타 상태이다. 사용하고자 하면 [github](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler#readme)를 확인하자
