---
layout: post
title:  "logging: ElasticSeach, kibana, fluentd"
date:   2019-12-18 01:02:59
name: k8s_log.md
category: kubernetes
---

- k8s같은 오케스트레이션 환경에서 로그를 수집할 때는 로그가 로컬 디스크에 쌓이지 않도록 해야한다(지정된 node로 들어가서 로그를 확인하기 힘들다)
- 실제로 간단하게 log를 확인하려면 **kubectl logs <pod이름>**
- aws 같은 cloud로 k8s를 사용하면 제공해준 로그 수집기를 사용하면 된다

ElasticSearch
=================

- Apache의 Java 오픈소스 분산 검색 엔진
- 많은 양의 데이터를 저장, 검색, 분석할 수 있음
- 주로 ELK(ElasticSearch, Logstash, Kibana)스택으로 사용
  - Logstash : 다양한 로그를 수집, 파싱하여 ElasticSearch로 전달
  - ElasticSearch : 데이터를 검색, 집계하여 필요한 정보를 얻는다. 여러대의 node에서 실행되도록 개발됨
  - Kibana : 데이터 시각화 및 모니터링 제공

- ES을 사용할 Deployment와 이를 다룰 service를 생성해보자

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  labels:
    app: elasticsearch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: elastic/elasticsearch:6.4.0
        env:
        - name: discovery.type
          value: "single-node"
        ports:
        - containerPort: 9200
        - containerPort: 9300

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch-svc
  namespace: default
spec:
  ports:
  - name: elasticsearch-rest
    nodePort: 30920
    port: 9200
    protocol: TCP
    targetPort: 9200
  - name: elasticsearch-nodecom
    nodePort: 30930
    port: 9300
    protocol: TCP
    targetPort: 9300  
  selector:
    app: elasticsearch
  type: NodePort
```

- crashloopBackoff 가 뜬 경우 [여기](https://managedkube.com/kubernetes/pod/failure/crashloopbackoff/k8sbot/troubleshooting/2019/02/12/pod-failure-crashloopbackoff.html)를 참고하자

- - -

kibana
======

- kibana를 elasticsearch랑 연동하자

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  labels:
    app: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: elastic/kibana:6.4.0
        env:
        - name: SERVER_NAME
          value: "kibana.kubenetes.example.com"
        - name: ELASTICSEARCH_URL
          value: "http://elasticsearch-svc.default.svc.cluster.local:9200"
        ports:
        - containerPort: 5601
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kibana
  name: kibana-svc
  namespace: default
spec:
  ports:
  - nodePort: 30561
    port: 5601
    protocol: TCP
    targetPort: 5601
  selector:
    app: kibana
  type: NodePort
```

- ElasticSearch의 IP는 **ELASTICSEARCH_URL** 를 통해서 전달된다(k8s의 DNS서비스를 이용한다)
- <외부주소>:30561을 브라우저에 띄어보자

- - -

fluentd
=========

- 로그 수집용 오픈소스 프로젝트

```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:elasticsearch
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch-svc.default.svc.cluster.local"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "http"
          - name: FLUENT_UID
            value: "0"
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

- DeamonSet을 사용하였기 때문에 노드에 자동으로 하나씩 pod들이 생성된다
- /var/log 에는 시스템용 프로세스의 로그가 쌓인다.
- /var/lib/docker/containers에는 pod에서 출력하는 로그가 쌓임
- FLUENT_UID=0 : 이 pod를 실행하는 유저에게 위의 두 로그가 쌓이는 디렉토리에 접근가능하게 한다
- 자세한 내용은 [여기](https://kubernetes.io/ko/docs/concepts/workloads/controllers/daemonset/)에서 확인하자

- 키바나 사용법은 [여기](https://www.elastic.co/guide/kr/kibana/current/getting-started.html)를 참고해서 여러가지 로그를 확인할 수 있다
