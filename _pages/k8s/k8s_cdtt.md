---
layout: post
title:  "cordon, drain, tain, toleration"
date:   2019-12-18 01:02:59
name: k8s_cdtt.md
category: kubernetes
---

cordon
======

- 지정된 **node** 에 **더 이상** (이미 있던 애들은 해당사항 아님) pod이 스케쥴링 되지 않는다

```
//cordon
kubectl cordon <node이름>

//uncordon
kubectl uncordon <node이름>
```

![cordon]({{site.baseurl}}/post_img/{{page.name}}/cordon.png)

- cordon을 하면 Ready,SchedulingDisabled 가 된다

- - -

drain
=====

- node의 pod를 다른 곳으로 이동
- 순서
  1. 새로운 pod이 node에 스케쥴링 되지 않도록 설정(cordon을 한다고 보면 됨)
  2. 실행중이던 pod삭제. DaemonSet으로 실행중인 경우 실패, --ignore-daemonsets=true를 설정해서 DeamonSet의 pod를 제외하고 drain 할 수 있다
  3. controller로 만들어진 pod들은 다른 node에 알아서 설정되지만, 직접 만들어진 pod들은 그냥 삭제되기 때문에 실패한다(강제로 하고자하면 --force 옵션 사용)
  4. kubelet이 직접 실행한 static pod들은 삭제 되지 않는다
  5. 해당 node를 사용하고 싶으면 uncordon을 하면 된다

- - -

taint & toleration
====

- **taint** : cordon과 같은 역할.
- But 모든 pod를 막는 것은 아니고 **toleration** 을 이용하여 특정 pod의 스케쥴링은 허용
- 주로 node가 특정 역할(DB 등)을 하게 할 때 사용

- tain는 key,value,effect로 구성

```
//kubectl taint nodes <nodeName> <keyName>=<value>:<effect>
kubectl taint nodes ms1 key01=value01:NoSchedule
```

- taint가 걸려 있는 node에는 toleration의 key,value,effect가 모두 taint와 같은 pod들만 스케쥴링 된다(**operator** 가 **Equal** 인 경우에만..)

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: test-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      name: test-app
      labels:
        app: test-app
    spec:
      containers:
      - name: test-container
        image: test-image
        ports:
        - containerPort: 8080
      tolerations:
        - key: "key01"  
          operator: "Equal"
          value: "value01"
          effect: "NoSchedule"
```

- taint 해제는 다음의 방법으로 한다

```
kubectl tain nodes ms01 key:NoSchedule-
```

effect의 종류
-------------

- NoSchedule : toleration이 없으면 pod가 스케쥴링 되지 않는다. 이미 실행되던 pod에는 적용되지 않음
- PreferNoSchedule : NoSchedule과 비슷하지만, 클러스터 내의 다른 node들에 자원이 부족하면 pod들이 스케쥴링 된다
- NoExecute : NoSchedule에다가 기존의 pod들에도 tain&toleration 규칙을 적용해서 쫓아내거나 남기거나를 결정한다

operator
-----------

- Equal : key, value, effect가 모두 같은지 확인
- Exists : value를 확인하지 않음. key와 effect만 확인.
  - key, effect도 없는 경우 : 모든 taint를 무시
  - key만 있음 : 효과는 상관 없이 해당 key를 가지는 taint에 적용
