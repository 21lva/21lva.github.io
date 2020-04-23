---
layout: post
title:  "ConfigMap, Secret"
date:   2019-12-18 01:02:59
name: k8s_configMap.md
category: kubernetes
---

ConfigMap
==========

- 환경변수, 설정값등을 변수로 관리해서 Pod 생성시 사용할 수 있다
- 새로운 pod을 만들때, configMap을 이용하면 쉽게 관리되는 변수들을 key-value로 사용할 수 있다

Literal 방식
----------

- 간단하게 문자로 생성하는 방법
- **kubectl create configmap newCM --from-literal=language=java** 로 간단하게 key(language), value(java)인 configMap을 newCM의 이름을 갖도록 만들 수 있다

- 또는 아래와 같이 yml파일로 작성할 수 있다

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: newCM
data:
  language: java
```

- 위의 설정을 Deployment에서 사용하는 예시가 아래의 yml파일이다

```yml
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
        env:
        - name: LANGUAGE
          valueFrom:
            configMapKeyRef:
               name: newCM
               key: language
```

test-container에서는 process.env.LANGUAGE 로 사용할 수 있다

File 방식
-----

- 위의 방식말고도 file로 제공할 수 있다

- 아래는 profile.properties라는 파일의 내용이다

```txt
github=21lva
age=27
```

- 위의 deploy.yaml의 key: 다음 부분을 **profile.properties** 라고 하면 된다.
- 조심해야할 부분은 파일 내용 자체가 하나의 문자열로 인식되기때문에 github이 key가 되고 21lva가 value가 되지는 않는다

volume에 mount해서 사용하기
---------------------

- 위의 configMap을 /tmp/config/ 에 mount해서 사용할 수 있다

```yml
#위는 생략
ports:
- containerPort: 8080
volumeMounts:
  - name: config-profile
    mountPath: /tmp/config
volumes:
- name: config-profile
  configMap:
    name: cm-file
```

- 이렇게 하면 mount된 volume이 configMap이 된다
- 내부의 파일들의 이름이 key가 된다


- - -

secret
------

- configMap에 민감한 정보를 넣는 것은 위험
- 비밀번호, ssh키 같은 정보는 **secret** 을 이용하여 저장하고 관리, 사용한다
- built-in secret : 클러스터 내부에서 API에 접근할 때 사용. ServiceAccount를 생성하면 자동으로 관련 secret 만들어지고 사용
- 사용자 시크릿 : **kubectl create secret** 또는 yml을 통해서 만들어진 secret. 메모리에 저장된다. 많이 만들면 메모리 이슈를 야기할 수 있음

- secret을 만들기 위해서는 **base64** 로 인코딩해서 사용해야 한다(실제로 사용할때 디코딩 한다). 이는 SSL인증서 같은 바이너리 파일을 저장하기 위해서 이다(이 파일을 문자열로 저장할 수 없다)

```
//리눅스에서 인코딩하는 방법
echo <원하는값> | base64
```

- 아래는 명령어로 만든 secret

```
//파일 생성
echo -n "username" > ./username.txt

//secret 생성
kubectl create secret generic user-secret --from-file=./username.txt

//secret 확인
kubectl get secret -o yaml
```

![secret]({{site.baseurl}}/post_img/{{page.name}}/secret.png)

인코딩된 값을 확인할 수 있다

```
//decoding
echo dXNlcm5hbWU= | base64 --decod
```

- 또는 다음의 yaml 파일을 만들어서 secret을 정의할 수 있다

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: user-secret
data:
  language: dXNlcm5hbWU=
```

- secret은 다음과 같이 사용가능


```yml
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
        env:
        - name: LANGUAGE
          valueFrom:
            secretKeyRef:
               name: newCM
               key: language
```

사용법은 configMap과 거의 비슷하다. 컨테이너 내부에서도 **process.env.<name>** 으로 사용가능하다

- volume으로 마운트 하는 것도 configMap과 거의 비슷(생략!)
