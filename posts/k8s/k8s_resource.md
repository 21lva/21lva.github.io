---
layout: post
title:  "리소스 관리"
date:   2019-12-18 01:02:59
name: k8s_resource.md
category: kubernetes
---


Pod의 Resource 관리
=======

CPU, Memory
-----------

- k8s에서는 컨테이너 단위로 리소스(CPU, memory 등) 제한 가능
- requests: 컨테이너가 필요로 하는 리소스 양(최소값)
- limits: 컨테이너가 사용할 수 있는 양(최대값). 실제로 pod에 limit값만큼의 리소스가 없더라도 스케쥴링 될 수 있기 때문에 조심해야한다. limits의 전체 합은 실제 node의 자원량보다 많을 수 있다(over-committed)

- cpu는 압축 가능한 리소스이기 때문에 limits보다 높은 양을 사용하려 하면 그 보다 많이 사용할 수는 없게 할 수 있다. 그러나 memory는 압축 불가능하기 때문에 프로세스가 limits보다 높은 memory를 요구하면 OOMKilled 된다. OOMKilled되면 k8s에서 pod를 재시작하지만, 지속적으로 OOMKilled되면 pod가 재시작 시간을 지연시키고 우리는 상태를 CrashLoopBackOff로 설정되는 것을 볼 수 있다.
- requests만 설정된 경우 limits가 자동으로 설정되지 않고 호스트 측의 리소스(특히 메모리)를 계속 소비하려 해서 OOM이 날 경우가 있으며, limits만 설정한 경우에는 limits 값을 reqeusts로 자동 설정된다. -> 둘 다 설정하도록 하자
- 둘 다 설정하지 않을 경우에는 최악의 경우 cpu를 전혀 할당 받지 않을 수도 있기 때문에 우선순위가 높은 컨테이너는 꼭 requests, limits를 설정하도록 하자


- 아래의 manifest로 deployment를 생성하자

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-resource
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
      - name: container
        image: busybox:latest
        command: ["dd","if=/dev/zero","of=/dev/null"]
        resources:
          requests:
            memory: "1024Mi"
            cpu: "500m"
          limits:
            memory: "2048Mi"
            cpu: "1000m"
```

- cpu 사용률을 확인하자

```
$ kubectl --kubeconfig=$KUBE_CONFIG get pods
NAME                              READY   STATUS      RESTARTS   AGE
Mem: 7797280K used, 333968K free, 87256K shrd, 360620K buff, 5897084K cached
CPU: 30.2% usr 23.6% sys  0.0% nic 45.8% idle  0.1% io  0.0% irq  0.1% sirq
Load average: 0.69 0.30 0.22 5/895 16
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     R     1308  0.0   0 50.1 dd if /dev/zero of /dev/null
   12     0 root     R     1316  0.0   1  0.0 top
```

- node의 cpu는 아래와 같이 2Gi개이고, requests와 limits 정보를 확인할 수 있다.

```bash
kubectl --kubeconfig=$KUBE_CONFIG describe nodes/nks-sdfasdfsf-w-mgx
Name:               nks-sdfasdfsf-w-mgx
Capacity:
  cpu:                2
  ephemeral-storage:  49563892Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             8131248Ki
  pods:               110
Allocatable:
  cpu:                1900m
  ephemeral-storage:  45678082792
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             7766704Ki
  pods:               110

Non-terminated Pods:          (14 in total)
  Namespace                   Name                                         CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                         ------------  ----------  ---------------  -------------  ---
  default                     sample-resource-65ffc597d5-pmnth             500m (26%)    1 (52%)     1Gi (13%)        2Gi (27%)      2m51s

Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                920m (48%)    1 (52%)
  memory             1364Mi (17%)  2388Mi (31%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
Events:              <none>
```

- 아래와 같이 변경해서 새로 배포해보자.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-resource
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
      - name: container
        image: busybox:latest
        command: ["dd","if=/dev/zero","of=/dev/null"]
        resources:
          requests:
            memory: "1024Mi"
            cpu: "200m"
          limits:
            memory: "2048Mi"
            cpu: "500m"
```

- 다시 노드를 살펴보면 requests와 limits가 변한 것을 볼 수 있다.

```bash
Non-terminated Pods:          (14 in total)
  Namespace                   Name                                         CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                         ------------  ----------  ---------------  -------------  ---
  default                     sample-resource-fbd554d6-4n292               200m (10%)    500m (26%)  1Gi (13%)        2Gi (27%)      67s
```

- top명령어로 확인해보면 cpu 사용량이 줄어든 것을 볼 수 있다

```
$ kubectl --kubeconfig=$KUBE_CONFIG exec -it pods/sample-resource-fbd554d6-4n292 top
Mem: 7797960K used, 333288K free, 87248K shrd, 360632K buff, 5897380K cached
CPU: 19.0% usr 14.2% sys  0.0% nic 66.6% idle  0.0% io  0.0% irq  0.0% sirq
Load average: 1.19 1.08 0.68 3/872 17
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     R     1308  0.0   1 23.7 dd if /dev/zero of /dev/null
   12     0 root     R     1316  0.0   0  0.0 top
```

- pod를 node에 스케쥴링할 때는 실제 사용량이 아닌 requests에 따라 할당된다


- 컨테이너에 메모리 설정을 할 때 주의할 사항이 있다. 컨테이너테 자원 설정을 하더라도 컨테이너는 그 양을 인식하지 못한다. 그래서 특정 프로세스의 경우 OOMKilled가 될 수 있다.
  - 예를 들어, java의 -Xms로 최대 힙을 설정하지 않으면 자바 app은 호스트의 메모리를 기준으로 최대 힙 크기를 설정하고 이 설정이 컨테이너의 메모리 제한보다 커지게 되어서 OOMKilled가 될 수 있다. -> JVM 1.8 이후부터는 cgroup으로 할당한 컨테이너 메모리 기준으로 설정할 수 있고, JVM 1.10 이후로는 **UserContainerSupport**로 제한을 설정할 수 있다
- CPU도 메모리와 비슷하다. 컨테이너에서는 호스트의 모든 컨테이너를 확인한다. CPU 제한이 하는 일은 컨테이너에 할당되는 CPU의 시간과 양이지 컨테이너가 바라보는 CPU를 제한하지는 않는다. CPU 수로 동작하는 프로세스의 경우에는 조심해야한다

Ephemeral storage
----------

- 컨테이너에서 사용하다가 사라져도 되는 데이터들은 컨테이너의 데이터 영역에 쌓아놓고 사용해도 되지만, 의도치 않은 상황에서는 노드의 디스크 영역을 과도하게 사용해서 문제를 야기할 수 있다. 이를 방지하기 위해서 ephemeral 스토리지에 제한을 설정해야하는 경우가 있다.
- k8s에서는 노드의 kubelet을 통해서 디스크 사용량을 주기적으로 확인한 후에 evict를 한다. kubelet은 컨테이너의 로그(kubectl logs로 확인가능한 로그들), emptyDir의 데이터(컨테이너가 같은 emptyDir을 쓰는 경우에는 그 둘의 합을 requests와 limits로 생각한다), 컨테이너의 쓰기 layer에 저장된 데이터를 ephemeral로 생각한다

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-resource
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
            cpu: "300m"
      - name: container2
        image: busybox:latest
        command: ["dd","if=/dev/zero","of=/dev/null"]
        resources:
          requests:
            memory: "1024Mi"
            cpu: "100m"
```

- - -

LimitRange
=========

- limitRange: pod, container, persistent volume cliam에 설정 해서 리소스의 최소값, 최대값, 기본값을 설정. 기존에 존재하는 것이 아닌 신규로 생성할 때만 적용된다.
- 모든 컨테이너에 requests와 limits를 설정하는 대신 컨테이너의 각 리소스에 최소/최대 값을 지정. 리소스 요청을 명시하지 않은 컨테이너에도 기본 값을 설정한다
- LimitRange는 LimitRanger라는 control plugin에 의해서 사용된다. pod의 manifest가 API 서버에 전달되면 LimitRanger가 pod의 spec을 검증한 후에 유효하지 않은 경우(ex. node보다 큰 pod)에는 거절한다.
- 하나의 네임스페이스에 개별 리소스에 제한을 거는 것이지만, 전체 총합에는 제한을 걸지 않는다
- Pod에 대한 제한은 컨테이너에서 사용하는 리소스 합
- PVC는 PV의 볼륨의 크기에 대한 제한

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: sample-lr
  namespace: default
spec:
  limits:
    - type: Container
      default:
        memory: 512Mi
        cpu: 500M
      defaultRequest:
        memory: 256Mi
        cpu: 250M
      max:
        memory: 1024Mi
        cpu: 1000M
      min:
        memory: 128Mi
        cpu: 200M
    - type: PersistentVolumeClaim
      max:
        memory: 1024Mi
        cpu: 1000M
      min:
        memory: 128Mi
        cpu: 200M
    - type: Container
      max:
        memory: 1024Mi
        cpu: 1000M
      min:
        memory: 128Mi
        cpu: 200M
```

- 설정 가능 항목은 다음과 같다
  - default: 기본 limits
  - defaultRequest: 기본 requests
  - max: 최대값
  - min: 최소값
  - maxLimitRequestRatio: Limits/Requests 비율

- 각 타입마다 설정할 수 있는 항목이 다르다
  - Container: default/ defaultReqeust/max/min/maxLimitRequestRatio
  - Pod: max/min/maxLimitRequestRatio
  - PVC: max/min

- - -

QoS
===

- requests와 limits 설정에 따라 QoS Class 값이 자동으로 설정된다.
- QoS class는 다음과 같다
  - Guaranteed: requests==limits이며 cpu,memory에 모두 설정. 우선순위 가장 높음
  - Burstable: Guaranteed 조건을 충족하지는 못하지만 requests또는 limits 중 최소한 하나가 설정되어 있음. 우선순위 중간
  - BestEffort: reqeusts/limits 모두 미지정. 우선순위가 가장 낮음. LimitRange가 설정되면 BestEffort가 되지 않는다
- QoS class는 oom score를 설정할때 사용됨
  - oom score: OOM Killer에 의해 프로세스가 정지 시킬 우선순위 값. -1000(최고 순위. 가장 kill이 안됨)~1000(최저 순위)로 설정됨.
  - Guaranteed의 oom score: -998. k8s의 시스템 구성요소(oom -999)외에 가장 순위가 높다
  - Burstable의 oom score: min(max(2,1000-1000*(메모리 requests)/메모리 용량), 999). 할당 받은 메모리에 비해 메모리 리퀘스트가 많을수록(실제 메모리를 많이 쓴다는 뜻) 우선순위가 높다.
  - BestEffort의 oos score: 1000
- 모든 pod을 Guaranteed로 하면 부하 증가에 따른 다른 pod로의 영향을 피할 수 있음. But 집약률이 낮아짐


- - -

ResourceQuota
=========

- limitRange는 개별 pod에 적용되지만, ResourceQuota는 각 네임스페이스에서 사용할 전체 리소스 양을 제한할 수 있음
- 각 네임스페이스마다 사용 가능한 리소스 제한 가능
- 이미 생성된 리소스에는 영향을 주지 않음

- 생성 가능한 리소스 수 제한을 할 수 있다

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: sample-rq
  namespace: default
spec:
  hard:
    count/configmaps: 10
    count/deployments.apps: 3
```

- 아래와 같이 리소스 사용량을 제한할 수도 있다

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: sample-rq
  namespace: default
spec:
  hard:
    requests.memory: 2Gi
    requests.storage: 5Gi
    limits.cpu: 4
```

- ResourceQuota로 설정을 걸면 해당 리소스가 그 항목들에 설정을 걸어야 한다. 그렇지 않으면 가동되지 않는다

- - -

리소스 사용량 모니터링
========

- requests, limits 등을 설정하기 위해서 리소스 사용량을 모니터링 하는 것은 중요하다.
- 각 노드에 배포되어 있는 kubelet에는 cAdvisor라는 agent가 있음
- cAdvisor: 개별 컨테이너와 노드 전체의 리소스 사용량을 수집.
- heapster: node 중 하나에서 pod로 실행. cAdvisor로부터 데이터를 수집해서 노출 시킨다.

- 아래는 nks에 배포되어있는 heapster의 명령어인 **top**을 이용하여 **현재의** node와 pod의 자원 사용률을 확인하는 것이다

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG top node
NAME                  CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
nks-sdfasdfsf-w-mgx   1060m        55%    2733Mi          36%
 $ kubectl --kubeconfig=$KUBE_CONFIG top pod --all-namespaces
NAMESPACE       NAME                                        CPU(cores)   MEMORY(bytes)
default         sample-hpa-deployment-8645b7497-p5xbw       974m         0Mi
default         sample-hpa-deployment-8645b7497-sm5fs       975m         0Mi
ingress-nginx   ingress-nginx-controller-66dc9984d8-nclpz   1m           69Mi
...
```

- cAdvisor은 짧은 기간동안의 데이터만을 보관하기 때문에 그보다 오래된 데이터를 수집하고자 하려면 Prometheus와 같은 다른 서비스를 이용해야 한다
