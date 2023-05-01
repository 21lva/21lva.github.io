---
layout: post
title:  "Volume"
date:   2021-09-28 01:02:59
name: k8s_volume.md
category: kubernetes
---

- volume: 사용 가능한 볼륨. manifest에서 직접 지정하여 사용할 수 있다. k8s에서 볼륨을 생성하거나 삭제하는 것이 불가능. 호스트 volume, nfs, Ceph등을 지정 가능
- persistent volume: 외부 영구 볼륨을 제공하는 시스템을 통해서 볼륨을 제공받는다. 새로운 볼륨을 생성하거나 샂게하는 작업이 가능.
- persistent volume claim: 생성된 persistent volume을 할당하는 리소스. 실제 pod에서 사용하기 위해서는 persistent volume claim으로 생성된 persistent volume을 할당 받아야 한다

- - -

Volume
======

- Pod의 구성 요소로써 manifest에서 정의된다.
- 생성, 삭제 될 수 없음
- 마운트가 되었다는 전체하에 Pod의 모든 컨테이너에서 접근 가능.
- Volumed을 초기화하거나 외부 소스의 내용으로 채우거나 마운트하는 과정은 Pod의 container가 시작되기 전에 수행된다
- 종류에 따라 volume이 지속적으로 유지되어서 pod 종료 후에도 다른 pod에서 사용 가능

- secret과 configmap도 사실은 volume의 한 종류

emptyDir
-------

- 일시적인 데이터를 저장하는 데에 사용되는 빈 디렉토리
- pod의 모든 container에서 접근 가능. pod내의 여러 컨테이너의 파일 공유에 사용된다
- pod가 종료되면 삭제된다
- 호스트의 임의의 영역에 마운트 된다(어디로 마운트 될지 정할 수 없음)
- 호스트의 다른 영역에 접근 불가능하다

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  containers:

  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo debian 컨테이너에서 안녕하세요 > /pod-data/index.html"]
```

- 1번 컨테이너에 접속하면 **/usr/share/nginx/html**에 index.html이 있는 것을 확인할 수 있다

hostPath
--------

- 호스트의 특정 영역에 마운트한다. 호스트의 파일시스템에 접근해서 파일을 읽고 쓸 수 있다
- pod이 종료가 되어도 volume이 삭제되지는 않는다. 같은 node에 pod이 생성되면 이전의 데이터를 읽을 수 있다
- pod가 같은 node에 스케쥴링 된다는 보장이 없으므로 db로 사용하기에는 적합하지 않음
- 사용 가능 type: Directory, DirecotryOrCreate, File, Socket, BlockDevice

- 테스트에 사용할 kubernetes cluster는 node가 하나이다

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG get nodes
NAME                  STATUS   ROLES    AGE    VERSION
nks-sdfasdfsf-w-mgx   Ready    <none>   2d2h   v1.18.17
```

- 이 node에 hostPath 볼륨을 가지는 pod을 아래의 manifest로 마운트하자(pod의 /srv에 호스트의 /etc를 마운트한다)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-hostpath
spec:
  containers:
  - image: nginx:latest
    name: nginx-container
    volumeMounts:
    - mountPath: /srv
      name: hostpath-sample
  volumes:
  - name: hostpath-sample
    hostPath:
      path: /etc
      type: DirectoryOrCreate
```

- 아래와 같이 이미 호스트의 데이터들이 들어와 있다

```bash
kubectl --kubeconfig=$KUBE_CONFIG exec -it pod/sample-hostpath /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@sample-hostpath:/# ls /srv
X11 ...
```

- - -

Persistent volume
=========

- pod에서 실행 중인 app의 데이터를 유지하고 pod가 다른 node로 스케쥴링 되어도 그 데이터를 유지해야 할 때 사용한다
- 모든 node에서 접근 가능해야 하기 때문에 NAS(Network attached storage)에 저장된다. NAS에는 다음과 같은 유형이 있다
  - GCE Persistent Disk
  - AWS Elastic Block Store
  - NFS
  - Ceph
  - CSI


CSI(Container Storage Interface)
-----

- container ochestration engine과 storage system을 연결해주는 interface

- - -

Persistent volume claim
==============

- PV를 요청하는 리소스
- 개발자가 app을 위해 PV를 필요로 하면 k8s에서 요구한 PV를 생성해서 전달한다.
- PV를 개발자가 직접 생성, 관리 하는 것은 어렵기 때문에 k8s 클러스터 관리자가 미리 정해놓은 기반 스토리지를 k8s API 서버가 PV로 생성해서 등록한다

- 개발자가 PVC를 생성 -> k8s에서 PVC를 미리 생성되어 있는 PV에 바인딩 -> 개발자가 PVC를 참조하는 볼륨을 갖는 pod을 생성
- 바인딩된 PV는 다른 사용자가 동시에 claim할 수 없다

- 아래와 같이 PVC를 생성한다

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: csi-pod-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: nks-block-storage
```

- PVC의 상태를 확인하자

```bash
$ kubectl --kubeconfig=$KUBE_CONFIG get pvc
NAME               STATUS   VOLUME                           CAPACITY   ACCESS MODES   STORAGECLASS        AGE
csi-pod-pvc        Bound    pvc-737dd0028ad64ecdb0ceaadc57   10Gi       RWO            nks-block-storage   41m

$ kubectl --kubeconfig=$KUBE_CONFIG get pv
NAME                             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS        REASON   AGE
pvc-90886bb6e258473ea033d450e2   10Gi       RWO            Delete           Bound    default/csi-pod-pvc        nks-block-storage            9s
```

- Access Modes는 다음과 같은 값을 갖는다
  - RWO(ReadWriteOnce): 하나의 node만이 읽기/쓰기용으로 볼륨 마운트 가능
  - ROX(ReadOnlyMany): 다수의 node가 읽기용으로 볼륨 마운트 가능
  - RWX(ReadWriteMany): 다수의 node가 읽기/쓰기용으로 볼륨 마운트 가능

- 아래의 manifest로 pod을 생성한다

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/data"
        name: my-volume
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: csi-pod-pvc
```
