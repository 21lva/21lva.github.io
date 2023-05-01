---
layout: post
title:  "Controller, 배포"
date:   2019-12-18 01:02:59
name: k8s_controller.md
category: kubernetes
---


Replication controller
=================

- 가장 기본적인 컨트롤러
- 지정된 숫자만큼 항상 pod들이 유지되도록 한다

```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

- *kubectl get pods* 로 확인하면 3개의 pod이 실행되고 있는 것을 확인할 수 있다

- Selector : 가져올 pod들의 label 기술
- Replicas : 유지되어야 하는 Pod의 수
- template : Pod에 대한 정보가 들어 있음. template의 정보가 다르면 RC가 나중에 동작하더라도 이전의 pod은 삭제하지 않음(template에 맞는 pod들만 다룬다는 얘기)

- RC삭제를 하게되면 pod들도 같이 삭제됨. **--cascade=false** 옵션을 주면 pod는 남기고 RC만 삭제 가능(**kubectl delete rc <RC이름> --cascade=false** )

- 특정 pod를 RC에서 떼어 놓으려면 edit을하자(**kubectl edit pod <pod이름>** ). label의 이름을 바꾸면 된다

rolling update
---------------

- 새로운 버전의 배포가 필요하면, 새 버전의 pod를 모두 바꾸는 것이 아닌, 하나씩 순차적으로 바꾸는 과정
- **kubectl rolling-update <pod이름> -f <rc이름> [--image=원하는 이미지]**

- - -

ReplicaSet
===========

- ReplicationController와 달리 **set based selector** 를 이용 가능. [관련 공식 문서](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement)

```yml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
    - {key: service, operator: Exists, values: [user]}
    - {key: service, operator: DoesNotExist, values: [db]}
```

- - -

Deployment
===========

- stateless app을 배포할 때 일반적으로 사용하는 컨트롤러
- ReplicaSet을 관리하여 롤링 업데이트, 롤백 등을 가능하게 하는 리소스이다
- Deployment가 ReplicaSet을 관리하고, ReplicaSet가 Pod들을 관리한다
- Deployment가 새로운 ReplicaSet을 생성하는 조건: spec.template에 변경이 있을 경우

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

배포 방식
--------

- 배포 방식에는 블루그린, 롤링업데이트, canary 등이 존재

#### 1) 롤링 업데이트

- ReplicationController를 직접 이용하는 방법
  1. 기존 RC1(replica controller)의 replicas을 -1 한다
  2. 새로운 RC2의 replicas를 +1 한다
  3. 반복한다
  4. 배포 중간에 잘못된 경우 위의 방법을 반대로 한다

- RC를 직접 이용하는 방법은 귀찮고 복잡하다
- Deployment은 내부적으로 replicaSet을 운영하기때문에 2개의 RS를 만드는 방식으로 롤링 업데이트를 진행할 수 있다
- 방법은 여러가지가 있다
  1. **kubectl set image deployment <deploy이름> app-label=<새로운 이미지>** 로 새 이미지를 적용
  2. **kubectl edit deploy <deploy이름>** 을 통해서 설정 정보를 열고 수정
  3. 새로운 yaml 파일을 만들거나 수정하고 **kubectl replace -f [yaml,yml파일]** 을 통해서 업데이트

배포 롤백
-------

- Deployment는 롤백 기능을 제공. 사용 중인 레플리카셋 이전의 레플리카 셋을 replica=0으로 관리하고 있다가 replica를 증가시키는 방식으로 진행한다

- 변경 내역은 **kubectl rollout history deploy nginx-deployment**
- **--revision=<번호>** 를 적으면 자세한 내용을 볼 수 있다

![deploy]({{site.baseurl}}/post_img/{{page.name}}/deploy.png)

- 직전 버전으로 돌아가고 싶으면 **kubectl rollout undo deploy <deploy이름> --to-revision=<번호>**

- **kubectl scale deploy <이름> --replicas=<숫자>** : pod개수 조정
- **kubectl rollout pause deploy <deploy이름>** : 배포 주이
- **kubectl set image deploy <이름> <app이름>=<image이름>** : 이미지 변경
- **kubectl rollout resume deploy <이름>** : 재배포

배포 상태
-------

- 배포를 하게되면 status의 변화가 발생
- **kubectl rollout status deploy <이름>** 으로 상태 확인

- Progressing
  - 새로운 replicaSet을 만들고 있을 때
  - 새로운 replicaSet의 pod개수를 늘리고 줄이고 있을 때
  - 새로운 pod가 준비상태가 되거나 이용가능한 상태일 때

- complete : 다음 조건들을 확인하여 배포 상태를 확인.
  - deployment가 관리하는 모든 replicaSet의 업데이트가 완료되었을 때
  - 모든 replicaSet가 사용가능할 때
  - 이전 replicaSet가 종료되었을 때

- - -

DaemonSet
============

- 레플리카셋의 특수한 형태
- 클러스터 전체의 node에 pod를 하나씩 올릴 때 사용
- 특정 node에만 배포도 가능. taint, toleration 옵션 사용. taint가 지정된 노드에는 pod가 배치되지 않고, toleration을 사용하면 그 node에 pod를 배치할 수 있다
- 보통 모니터링, 로그 수집 용도로 사용

```yaml
#sample-ds.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: sample-ds
spec:
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
```

Update
-----

- 데몬셋을 업데이트 하는 방법에는 2가지가 있다
  - OnDelete: 메니페스트 변경시 업데이트 하는 것이 아니라, pod을 다시 생성할 때 새로운 메니페스트 정보로 pod을 생성
  - RollingUpdate: 하나씩 변경


- - -

Job
===

- 한 번 시행되고 끝나는 형태의 작업에 사용
- 특정 개수의 pod이 성공적으로 완료되는 것을 보장
- Job을 정의할때는 보통 image뿐만 아니라 컨테이너에서 Job을 수행하기 위한 **command** 도 필요

종류
---

- 단일 : pod 하나만 실행. **spec.completions** 와 **spec.parallelism** 을 설정하지 않는다(기본 1)
- 완료 개수가 있는 parallel job : **spec.completions** 에 양수를 설정.
- work queue를 가진 parallel job
  - **spec.completions** 은 설정하지 않고, **spec.parallelism** 은 양수로 설정
  - **spec.completions** 을 설정하지 않으면, **spec.parallelism** 과 같은 수로 설정됨
  - 각 pod들은 종료됐는지를 모두 독립적으로 결정할 수 있고, 모든 job이 실행
  - 최소한 1개의 pod가 성공적으로 종료되고 모든 pod가 종료되면 job이 성공적으로 종료됨

- - -

cron job
========

- job을 시간 기준으로 관리
- 지정된 시간에 한번만 job을 실행하거나 주기적으로 지정된 시간동안 반복하면서 job을 실행

- - -

StatefulSet
==========

- 레플리카셋의 특수한 형태. 차이점은 파드 명이 변경되지 않는다는 것 + 영구적으로 데이터 저장
- stateless app이 아닌 state가 있는 app을 관리하기 위함
- volume을 사용해서 특정 데이터를 기록해두고 포드가 재시작했을때고 유지할 수 있음

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sample-sfs
spec:
  serviceName: sample-sfs
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1G
```

- 위의 메니페스트를 이용하여 StatefulSet을 만들고 **persistent volume claim**을 확인해보자

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG get persistentvolumeclaims
NAME               STATUS   VOLUME                           CAPACITY   ACCESS MODES   STORAGECLASS        AGE
www-sample-sfs-0   Bound    pvc-5de11b7b207442f580e89d62ee   10Gi       RWO            nks-block-storage   55s
www-sample-sfs-1   Bound    pvc-db4099b18b554ded84eaa3754f   10Gi       RWO            nks-block-storage   40s
www-sample-sfs-2   Bound    pvc-f98a35378399436badcd375452   10Gi       RWO            nks-block-storage   19s
```

- StatefulSet은 replicaSet이나 DaemonSet과 달리 한 번에 하나의 pod만 생성, 삭제 한다(spec.podManagementPolicy=Parallel로 하면 병렬로 처리한다. 기본값은 OrderedReady)
   - 삭제 순서는 pod뒤의 숫자가 큰 것부터 삭제. 숫자가 클 수록 가장 최근에 생성된 pod이다. 이를 이용하면 master-slave형태의 StatefulSet을 구성할 수 있다

업데이트
-----

- 데몬셋과 비슷하게 업데이트는 RollingUpdate와 OnDelete로 나뉜다
- OnDelete: 메니페스트 변경이 아닌 pod 생성할 때 변경사항을 적용하는 업데이트 방식.
- RollingUpdate: StatefulSet은 persistent 데이터가 존재하므로 pod를 생성해서 하는 롤링 업데이트를 할 수 없다. StatefulSet에서는 partition을 설정할 수 있다
  - partition: 전체 pod중 설정된 partition 값 미만의 인덱스를 가지는 pod들은 업데이트 하지 않는다. 이 숫자를 하나씩 줄여나가면서 하나씩 업데이트를 진행해야한다
