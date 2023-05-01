---
layout: post
title:  "Docker - Container"
date:   2019-12-18 01:02:59
author: Inhyuk
category: Docker
tags: docker
cover:  "/assets/instacode.png"
name: docker0.md
---

- 기존의 Virtual Machine(VM)은 운영체제 위에 새로운 운영체제를 동작시키기 때문에 리소스 적인 측면에서 매우 많은 손해가 있다(보안의 장점이 있긴 한다)
![docker vs VM]({{site.baseurl}}/post_img/{{page.name}}/vm-vs-docker.png)
[이미지 출처](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
- 도커는 컨테이너를 이용하여 프로세스 단위로 가상화를 한다.
- 도커에 새로운 ubuntu 컨테이너를 사용한다고 해도 새로운 kernel이 생기는 것은 아니다. 관련된 내용은 [shared kenrel of docker](https://stackoverflow.com/questions/32756988/what-is-meant-by-shared-kernel-in-docker)를 참고하자.
- 이전에는 LXC(리눅스 컨테이너)를 사용했지만 현재는 libcontainer를 사용한다
- 이미지 : 컨테이너 실행에 필요한 의존성 파일, 설정 값등을 포함하고 있다. 이를 이용하여 새로운 컨테이너를 생성하기도 하고 배포하기도 한다. url 방식으로 관리한다.
- 레이어 : 이미지를 여러가지의 레이어로 분할하여 배포, 다운을 한다. 이미지의 특정 부분이 수정되었을 경우에 그 부분의 레이어만 다시 받으면 된다. [도커 기본 이미지란 무엇인가](https://www.quora.com/What-exactly-is-a-base-image-in-Docker)
- 도커의 이미지를 배포하게 되면 의존성을 신경쓸 필요 없다. 새로운 서버를 만들게 되면 만들어 놓은 이미지를 다운 받아서 컨테이너를 생성하기만 하면 된다.
- 배포에 이용하는 도커 이미지는 [docker hub](https://hub.docker.com/)에서 다운받아 사용할 수 있다.
- Dockerfile : 이미지 생성 과정을 적는 파일. 설치, 의존성 같은 부분을 적어서 쉽게 수정하고 관리할 수 있게 해준다.

- - -

격리 기능
========

- 도커 컨테이너는 vm과 다르다. vm은 호스트의 OS위에 하드웨어를 emulation하고 또 그 위에 OS를 올린다. 도커는 하드웨어 emulation이 없고 호스트의 커널을 공유해서 프로세스를 실행시킨다.
- [참고 자료](https://www.44bits.io/ko/keyword/linux-container#%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90%EC%84%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EA%B2%A9%EB%A6%AC-%EA%B8%B0%EB%8A%A5)
- 도커 컨테이너는 네임스페이스, chroot, cgroup 등을 이용하여 컨테이너끼리, 컨테이너와 호스트의 격리를 한다.
- 컨테이너는 리눅스 커널에 포함된 여러 격리 기술을 사용하여 생성된 **프로세스**이다

Namespace
---------

- linux namesapce: 특정 프로세스의 리눅스 리소스 접근을 제어하기 위한 기능. mnt(독립적인 파일시스템), pid(독립적인 프로세스 공간), net(network간의 충돌 방지), ipc(프로세스간의 독립적인 통신), uts(독립적인 hostname), user(독립적인 사용자 할당) 등이 존재.
- 기본적으로 모든 프로세스는 init 프로세스의 네임스페이스를 공유하지만 **unshare** 명령어로 네임스페이스를 분리할 수 있다.

#### Pid Namespace

- PID namespace: 프로세스의 ID를 격리하는 네임스페이스. 컨테이너가 init 프로세스인 것처럼 실행한다.
- 리눅스에서는 **/proc** 디렉토리에 프로세스에 대한 정보가 있다. **/proc** 밑에는 숫자(pid)로된 디렉토리들이 존재하고 이 디렉토리에는 **ns**라는 디렉토가 있어서 다양한 네임스페이스 정보를 포함한다.
- 컨테이너에서는 하나의 프로세스이며 그 pid namespace안에서는 pid 1번을 가진다.

- 도커를 설치하고 다음의 명령어를 입력해보자

```
//호스트
[root@s181376e15b1 ~]# uname -a
Linux s181376e15b1 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

[root@s181376e15b1 ~]# docker run -it ubuntu:latest bash
root@7e80d6b9f599:/# uname -a
Linux 7e80d6b9f599 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
root@7e80d6b9f599:/# cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
~생략~

[root@s181376e15b1 ~]# docker run -it centos:latest bash
[root@99d40e106d59 /]# uname -a
Linux 99d40e106d59 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
[root@99d40e106d59 /]# cat /etc/*-release
CentOS Linux release 8.4.2105
NAME="CentOS Linux"
~생략~
```

- 위의 결과에서 다음의 내용을 알 수 있다.
  1. *uname -a*는 커널의 정보를 보여준다. 도커 컨테이너들과 호스트는 같은 커널을 사용한다
  2. *cat /ect/*-release*는 런타임 환경을 보여준다. 도커는 각각의 환경이 다르다.
  3. 도커컨테이너는 가상머신이 아니고 호스트에서 동작하는 **격리된 프로세스**이다.

```
[root@s181376e15b1 ~]# docker run -itd ubuntu
dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43
[root@s181376e15b1 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
dd23b01fdf07   ubuntu    "bash"    5 seconds ago   Up 4 seconds             upbeat_bassi
[root@s181376e15b1 ~]# ps -aux | grep dd23b01fdf07
root     18192  0.0  0.1 711736  7164 ?        Sl   13:30   0:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43 -address /run/containerd/containerd.sock
```

- 도커 없이 pid 1번으로 프로세스를 실행시켜보자. chroot를 쓰는 것은 복잡하니 이미지를 복사해서 사용한다.

```
[root@s181376e15b1 ~]# mkdir busybox-image
[root@s181376e15b1 ~]# docker run --name busybox-image busybox:latest
[root@s181376e15b1 ~]# docker export busybox-image > ./busybox-image/image.tar
[root@s181376e15b1 ~]# cd busybox-image; tar xf image.tar; cd ..
[root@s181376e15b1 ~]# sudo unshare -p -f --mount-proc chroot ./busybox-image /bin/sh
/ # echo $$
1  => 스스로를 pid 1이라고 인식하지만 호스트에서는 다른 pid로 인식한다.
```

- 지금 실행한 process의 namepsace의 정보는 */proc/{pid}/ns*에서 볼 수 있다. 격리된 프로세스의 pid namespace는 다른 pid ns랑 다르다.

```
[root@s181376e15b1 ~]# sudo unshare -p -f --mount-proc chroot ./busybox-image /bin/sh &
[root@s181376e15b1 ~]# ps -aux --sort +start_time | tail -n 5
root     20715  0.0  0.0 108044   592 pts/0    S    13:47   0:00 unshare -p -f --mount-proc chroot ./busybox-image /bin/sh
root     20716  0.0  0.0   1324   264 pts/0    S+   13:47   0:00 /bin/sh
root     20849  0.0  0.0      0     0 ?        S    13:48   0:00 [kworker/1:0]
root     20948  0.0  0.0 157676  2004 pts/1    R+   13:49   0:00 ps -aux --sort +start_time
root     20949  0.0  0.0 108152   696 pts/1    S+   13:49   0:00 tail -n 5
[root@s181376e15b1 ~]# ls -l /proc/20716/ns
합계 0
lrwxrwxrwx 1 root root 0  7월  2 13:50 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 root root 0  7월  2 13:50 mnt -> mnt:[4026532312]
lrwxrwxrwx 1 root root 0  7월  2 13:50 net -> net:[4026532028]
lrwxrwxrwx 1 root root 0  7월  2 13:50 pid -> pid:[4026532313]
lrwxrwxrwx 1 root root 0  7월  2 13:50 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0  7월  2 13:50 uts -> uts:[4026531838][root@s181376e15b1 proc]# ls -l 1/ns/
합계 0
lrwxrwxrwx 1 root root 0  7월  2 13:45 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 root root 0  7월  2 13:45 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 root root 0  7월  2 13:45 net -> net:[4026532028]
lrwxrwxrwx 1 root root 0  7월  2 13:45 pid -> pid:[4026531836]
lrwxrwxrwx 1 root root 0  7월  2 13:45 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0  7월  2 13:45 uts -> uts:[4026531838]
```

- 컨테이너는 호스트에서 실행 중인 프로세스이다.

```
//실행중인 컨테이너의 정보
[root@s181376e15b1 dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43]# pwd
/var/run/docker/runtime-runc/moby/dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43

//컨테이너의 pid
[root@s181376e15b1 dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43]# cat state.json | jq '.init_process_pid'
18211

//컨테이너의 네임스페이스 정보
[root@s181376e15b1 dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43]# cat state.json | jq '.namespace_paths'
{
  "NEWIPC": "/proc/18211/ns/ipc",
  "NEWNET": "/proc/18211/ns/net",
  "NEWNS": "/proc/18211/ns/mnt",
  "NEWPID": "/proc/18211/ns/pid",
  "NEWUSER": "/proc/18211/ns/user",
  "NEWUTS": "/proc/18211/ns/uts"
}

//docker container를 기존 프로세스 다루듯 kill 해보자.
[root@s181376e15b1 dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43]# ps -aux | grep 18211
root     18211  0.0  0.0   4476  2092 pts/0    Ss+  13:30   0:00 bash
root     22858  0.0  0.0 116980  1024 pts/1    S+   14:04   0:00 grep --color=auto 18211
[root@s181376e15b1 dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43]# kill -9 18211
[root@s181376e15b1 dd23b01fdf070aa44bfba4cd5eb7d2adc506e35bbaea4143e5e6549a32b3dd43]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```


#### UTS Namespace

- 네트워크 격리를 위해 사용되는 네임스페이스 중 하나
- 컨테이너에게 hostname을 격리하기 위해 사용된다

- 아래의 방법으로 호스트의 호스트네임을 확인해보자

```bash
root@s17b59a580ca:~# hostname
s17b59a580ca
```

- 새로운 파일을 만들고 **unshare**를 통해서 UTS namespace를 새로 만든다. 그 후에 **nsenter**로 새로 생성된 파일의 hostname을 확인한다
  - nsenter: 격리된 namespace에 명령어를 실행시키는 명령어

```bash
root@s17b59a580ca:~# touch a
root@s17b59a580ca:~# ls
a
root@s17b59a580ca:~# unshare --uts=a hostname tmpUts
root@s17b59a580ca:~# nsenter --uts=a hostname
tmpUts
```

- 아래의 방법으로 실제로 네임스페이스가 분리되었는지 확인한다

```bash
root@s17b59a580ca:~# echo $$
73831
root@s17b59a580ca:~# nsenter --uts=a bash
root@tmpUts:~# echo $$
74245
```

- 다른 터미널로 접속해서 init 프로세스, 호스트의 bash 프로세스, 새로 격리한 bash 프로세스의 namespace정보를 확인해보자. uts namespace만 달라진 것을 확인할 수 있다.

```bash
root@s17b59a580ca:~# ll /proc/1/ns
lrwxrwxrwx 1 root root 0 Sep  7 11:32 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 net -> net:[4026532029]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 pid -> pid:[4026531836]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 uts -> uts:[4026531838]
root@s17b59a580ca:~# ll /proc/73831/ns
lrwxrwxrwx 1 root root 0 Sep  7 11:29 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 root root 0 Sep  7 11:29 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 root root 0 Sep  7 11:29 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 root root 0 Sep  7 11:29 net -> net:[4026532029]
lrwxrwxrwx 1 root root 0 Sep  7 11:29 pid -> pid:[4026531836]
lrwxrwxrwx 1 root root 0 Sep  7 11:29 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Sep  7 11:29 uts -> uts:[4026531838]
root@s17b59a580ca:~# ll /proc/74245/ns
lrwxrwxrwx 1 root root 0 Sep  7 11:32 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 net -> net:[4026532029]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 pid -> pid:[4026531836]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Sep  7 11:32 uts -> uts:[4026532311]
```

#### 네트워크 네임스페이스

- 네트워크 네임스페이스: 프로세스간의 네트워크 환경을 격리하는 기능 제공

- **ip**명령어를 사용하여 가상 인터페이스(veth, virtual ethernet device)를 만든다. veth는 항상 쌍으로 만들어진다.
- 현재의 모든 네트워크는 default namespace에 속해 있다.

- *ip link add** 명령어를 통해서 veth 한 쌍을 만들어 보자

```
root@s17b59a580ca:~# ip link add veth0 type veth peer name veth1
root@s17b59a580ca:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             f2:20:af:31:52:b1 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethef6fc9c@if32 UP             9a:dd:c9:2d:1c:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
veth1@veth0      DOWN           aa:7d:a7:96:b6:de <BROADCAST,MULTICAST,M-DOWN>
veth0@veth1      DOWN           fe:89:9d:47:db:87 <BROADCAST,MULTICAST,M-DOWN>
```

- **ip netns** 명령어로 새로운 namespace를 만들고, 해당 namespace에서 루프 인터페이스를 동작시킨다.

```
//net_ns라는 새로운 네트워크 네임스페이스 생성
root@s17b59a580ca:~# ip netns add new_ns
root@s17b59a580ca:~# ip netns list
new_ns
//새 네임스페이스에서 네트워크 디바이스 출력
root@s17b59a580ca:~# ip netns exec new_ns ip --br link
lo               DOWN           00:00:00:00:00:00 <LOOPBACK>
//새 네임스페이스의 루프백 네트워크 디바이스 UP
root@s17b59a580ca:~# ip netns exec new_ns ip link set dev lo up
root@s17b59a580ca:~# ip netns exec new_ns ip --br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
```

- nginx를 설치하고 종료한 후에, 새로운 namespace에서 실행시킨다. 그 후에 다른 터미널에서 기존의 namespace에서 연결되지 않는 것과 새로운 namespace에서 연결되는 것을 확인할 수 있다.

```
root@s17b59a580ca:~# apt install nginx-core
root@s17b59a580ca:~# systemctl stop nginx
root@s17b59a580ca:~# systemctl disable nginx
root@s17b59a580ca:~# curl 127.0.0.1
curl: (7) Failed to connect to 127.0.0.1 port 80: Connection refused
root@s17b59a580ca:~# ip netns exec new_ns nginx -g 'daemon off;'
root@s17b59a580ca:~# ip netns exec new_ns curl 127.0.0.1
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

- *LISTEN* 중인 port를 확인한다.

```bash
root@s17b59a580ca:~# netstat -nat | grep LISTEN
tcp        0      0 127.0.0.1:39370         0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
root@s17b59a580ca:~# ip netns exec new_ns netstat -nat | grep LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp6       0      0 :::80                   :::*                    LISTEN
```

- 이제 새로 만들어서 격리되어 있는 네트워크 네임스페이스와 호스트를 연결해보자. 이전에 만들어 둔 가상 인터페이스 중 하나를 새로 만든 network namespace로 이전시킨다. 그 후에 ip를 할당하고 두 가상 인터페이스를 UP으로 만든다

```bash
//veth1을 새 네임스페이스로 이동
root@s17b59a580ca:~# ip link set veth1 netns new_ns
root@s17b59a580ca:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             f2:20:af:31:52:b1 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethef6fc9c@if32 UP             9a:dd:c9:2d:1c:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
veth0@if34       DOWN           fe:89:9d:47:db:87 <BROADCAST,MULTICAST>
root@s17b59a580ca:~# ip netns exec new_ns ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
veth1@if35       DOWN           aa:7d:a7:96:b6:de <BROADCAST,MULTICAST>

//veth0에 10.200.0.2, veth1에는 10.200.0.3 할당. 호스트의 10.200.0.2/24의 패킷들은 전부 veth0을 통해서 veth1로 가고, 새로운 네임스페이스의 10.200.0.3/24 는 전부 veth1을 통해서 veth0으로 흘러간다
root@s17b59a580ca:~# ip a add 10.200.0.2/24 dev veth0
root@s17b59a580ca:~# ip netns exec new_ns ip a add 10.200.0.3/24 dev veth1

//veth0, veth1 활성화(UP)
root@s17b59a580ca:~# ip link set dev veth0 up
root@s17b59a580ca:~# ip netns exec new_ns ip link set dev veth1 up
root@s17b59a580ca:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             f2:20:af:31:52:b1 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethef6fc9c@if32 UP             9a:dd:c9:2d:1c:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
veth0@if34       UP             fe:89:9d:47:db:87 <BROADCAST,MULTICAST,UP,LOWER_UP>
root@s17b59a580ca:~# ip netns exec new_ns ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
veth1@if35       UP             aa:7d:a7:96:b6:de <BROADCAST,MULTICAST,UP,LOWER_UP>
```

- 이제 격리된 namespace에 있는 nginx로 연결할 수 있다(80 port는 안써도 자동으로 붙는다)

```
root@s17b59a580ca:~# curl 10.200.0.3:80
~ 생략 ~
<title>Welcome to nginx!</title>
~ 생략 ~
```

- veth을 직접 생성해서 만드는 것은 불편하니 **Bridge**라는 L2 계층 장비(segment들을 송수신)를 가상으로 만들어서 사용해보자. 아래의 명령어로 Bridge를 생성하고 UP으로 만든다.

```
root@s17b59a580ca:~# ip link add br0 type bridge
root@s17b59a580ca:~# ip link set br0 up
```

- 그리고 위에서 사용한 가상 인터페이스들(veth0, veth1)을 삭제해보자.

```
root@s17b59a580ca:~# ip link delete veth0
root@s17b59a580ca:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             f2:20:af:31:52:b1 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethef6fc9c@if32 UP             9a:dd:c9:2d:1c:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
br0              UNKNOWN        8a:86:7e:36:f5:03 <BROADCAST,MULTICAST,UP,LOWER_UP>
root@s17b59a580ca:~# ip netns exec new_ns ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
```

- bridge와 new_ns 네임스페이스에 가상 인터페이스를 생성한다. 아직 brdige의 가상 인터페이스를 활성화 시키지 않았기 때문에 UNKNOWN 상태이다.

```
//brid0이라는 가상 인터페이스와 그 짝인 veth0이라는 가상 인터페이스 생성
root@s17b59a580ca:~# ip link add brid0 type veth peer name veth0

//veth0을 새 네임스페이스에 등록
root@s17b59a580ca:~# ip link set veth0 netns new_ns

//veth0에 10.200.0.2/24 할당
root@s17b59a580ca:~# ip netns exec new_ns ip a add 10.200.0.2/24 dev veth0

//새 네임스페이스의 네트워크 디바이스들 활성화
root@s17b59a580ca:~# ip netns exec new_ns ip link set dev lo up
root@s17b59a580ca:~# ip netns exec new_ns ip link set dev veth0 up
root@s17b59a580ca:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             f2:20:af:31:52:b1 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethef6fc9c@if32 UP             9a:dd:c9:2d:1c:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
br0              UNKNOWN        8a:86:7e:36:f5:03 <BROADCAST,MULTICAST,UP,LOWER_UP>
brid0@if37       DOWN           16:32:b8:7c:d5:da <BROADCAST,MULTICAST>
root@s17b59a580ca:~# ip netns exec new_ns ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
veth0@if38       LOWERLAYERDOWN 36:a5:75:ce:94:ac <NO-CARRIER,BROADCAST,MULTICAST,UP>
```

- brid0 인터페이스를 만들어둔 bridge에 연결시키고 활성화 해준다.

```
//br0 브릿지에 bird0 등록.
root@s17b59a580ca:~# ip link set brid0 master br0
root@s17b59a580ca:~# ip link set dev brid0 up
root@s17b59a580ca:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             f2:20:af:31:52:b1 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethef6fc9c@if32 UP             9a:dd:c9:2d:1c:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
br0              UP             16:32:b8:7c:d5:da <BROADCAST,MULTICAST,UP,LOWER_UP>
brid0@if37       UP             16:32:b8:7c:d5:da <BROADCAST,MULTICAST,UP,LOWER_UP>
root@s17b59a580ca:~# ip netns exec new_ns ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
veth0@if38       UP             36:a5:75:ce:94:ac <BROADCAST,MULTICAST,UP,LOWER_UP>
```

- 새로운 network namespace를 만들고 위의 작업을 반복한다.

```
root@s17b59a580ca:~# ip netns add new_ns2
root@s17b59a580ca:~# ip link add brid1 type veth peer name veth1
root@s17b59a580ca:~# ip link set veth1 netns new_ns2
root@s17b59a580ca:~# ip netns exec new_ns2 ip a add 10.200.0.3/24 dev veth1
root@s17b59a580ca:~# ip netns exec new_ns2 ip link set dev lo up
root@s17b59a580ca:~# ip netns exec new_ns2 ip link set dev veth1 up
root@s17b59a580ca:~# ip link set brid1 master br0
root@s17b59a580ca:~# ip link set dev brid1 up
root@s17b59a580ca:~# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             f2:20:af:31:52:b1 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:4a:43:ba:72 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethef6fc9c@if32 UP             9a:dd:c9:2d:1c:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
br0              UP             16:32:b8:7c:d5:da <BROADCAST,MULTICAST,UP,LOWER_UP>
brid0@if37       UP             16:32:b8:7c:d5:da <BROADCAST,MULTICAST,UP,LOWER_UP>
brid1@if39       UP             62:7f:d5:ab:33:76 <BROADCAST,MULTICAST,UP,LOWER_UP>
root@s17b59a580ca:~# ip netns exec new_ns2 ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
veth1@if40       UP             e2:1f:3b:7e:0d:20 <BROADCAST,MULTICAST,UP,LOWER_UP>
```

- 새로 만든 namspace에서 다른 namespace에 있는 nginx에 연결해보자(그 전에 방화벽을 설정해야한다)

```
//FORWARD 규칙 확인. 활성화 되지 않은 상태.
root@s17b59a580ca:~# iptables -L | grep FORWARD
Chain FORWARD (policy DROP)
//FORWARD 규칙 활성화
root@s17b59a580ca:~# iptables --policy FORWARD ACCEPT
root@s17b59a580ca:~# iptables -L | grep FORWARD
Chain FORWARD (policy ACCEPT)
root@s17b59a580ca:~# ip netns exec new_ns2 curl 10.200.0.2:80
~ 생략 ~
<title>Welcome to nginx!</title>
~ 생략 ~
```

- bridge가 호스트의 같은 네임스페이스에 있지만, ip가 부여되어 있지 않기 때문에 호스트에서는 격리된 두 네임스페이스에 접근할 수 없다.

```
root@s17b59a580ca:~# curl 10.200.0.2:80
~ 아무 일 없음 ~
```

- **route** 명령어를 통해서 보면 10.200.0.2로 가야할 패킷을 어디로 보내야할지 모르는 것을 알 수 있다.

```
root@s17b59a580ca:~# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.0.0.1        0.0.0.0         UG    0      0        0 eth0
10.0.0.0        *               255.255.255.0   U     0      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
root@s17b59a580ca:~# ip netns exec new_ns2 route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.200.0.0      *               255.255.255.0   U     0      0        0 veth1
```

- br0에 ip를 부여하면 연결할 수 있다.

```
root@s17b59a580ca:~# ip a add 10.200.0.1/24 brd 10.200.0.255 dev br0
root@s17b59a580ca:~# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.0.0.1        0.0.0.0         UG    0      0        0 eth0
10.0.0.0        *               255.255.255.0   U     0      0        0 eth0
10.200.0.0      *               255.255.255.0   U     0      0        0 br0
172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
root@s17b59a580ca:~# curl 10.200.0.2:80
~ 생략 ~
<title>Welcome to nginx!</title>
~ 생략 ~
```

- 현재 상태에서는 호스트는 인터넷에 연결할 수 있지만, 격리된 네임스페이스는 그렇지 못하다. **route** 명령어를 통해서 보면 호스트와 달리 새로운 namespace에는 **default** 규칙이 없다.
- *(snowflake) 또는 0.0.0.0은 gateway없이 직접 접근 가능하다는 의미이다.[default gateway](https://superuser.com/questions/580684/why-there-is-only-one-default-gateway/580703#580703)

```
root@s17b59a580ca:~# ping google.com
PING google.com (172.217.25.78) 56(84) bytes of data.
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms

root@s17b59a580ca:~# ip netns exec new_ns ping google.com
ping: unknown host google.com

//host와 달리 네임스페이스에는 default가 설정되어 있지 않다.
root@s17b59a580ca:~# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.0.0.1        0.0.0.0         UG    0      0        0 eth0
10.0.0.0        *               255.255.255.0   U     0      0        0 eth0
10.200.0.0      *               255.255.255.0   U     0      0        0 br0
172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
root@s17b59a580ca:~# ip netns exec new_ns route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.200.0.0      *               255.255.255.0   U     0      0        0 veth0
```

- 새 네임스페이스에 default gateway를 추가해보자.

```
root@s17b59a580ca:~# ip netns exec new_ns ip route add default via 10.200.0.1
root@s17b59a580ca:~# ip netns exec new_ns route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.200.0.1      0.0.0.0         UG    0      0        0 veth0
10.200.0.0      *               255.255.255.0   U     0      0        0 veth0
```

- NAT 설정을 해보자

```
//NAT 설정을 위해서 ip forward 기능 활성화
root@s17b59a580ca:~# sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1

//iptables에 NAT 규칙 추가
root@s17b59a580ca:~# iptables -t nat -A POSTROUTING -s 10.200.0.1/24 -j MASQUERADE

//이제 새 네임스페이스에서 인터넷으로 연결이 가능하다.
root@s17b59a580ca:~# ip netns exec new_ns ping google.com
PING google.com (172.217.26.14) 56(84) bytes of data.
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
```

##### iptables

- 리눅스에서 방화벽을 설정하는 명령어. 커널의 netfilter의 packet filtering을 기능을 제공
- Chain: 패킷을 다루는 상태를 지정. ACCEPT(허용), DROP(차단)등으로 선택
  - Chain Input: 서버로 들어오는 패킷에 대한 정책
  - Chain FORWARD: 서버에서 forwarding(서버를 거쳐가는)되는 패킷에 대한 정책
  - Chain OUTPUT: 서버에서 나가는 패킷에대한 장척

```
root@s17b59a580ca:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DOCKER-USER  all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain DOCKER (1 references)
target     prot opt source               destination

Chain DOCKER-USER (1 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
```

#### chroot

- chroot: 프로세스가 동작하는 루트 디렉토리를 변경하는 시스템 콜
- 프로세스의 root directory를 지정하면 그 루트 디렉토리의 자손 디렉토리가 아닌 디렉토리들은 접근할 수 없으므로 프로세스간의 디렉토리 접근을 방지할 수 있다.
  - **jailbreaking**을 통해서 상위 디렉토리로 접근 가능하다.

- *chroo {루트} {명령어}*: root를 주어진 경로로 하고 명령어를 수행하는 프로세스를 시작하는 명령어. 지금 그대로 사용하면 바꾼 루트 디렉토리에서 명령어를 찾을 수 없다는 오류가 발생할 것이다(아래에서는 /tmp/bin/bash). 왜냐하면 바꾼 경로(/bin/bash)는 실제로는 /tmp/bin/bash이고 당연히 /tmp에는 /bin/bash가 없으니 오류가 나는 것이다.

```
//chroot 루트경로 프로세스
root@s17b59a580ca:/etc/network# chroot /tmp /bin/bash
chroot: failed to run command ‘/bin/bash’: No such file or directory
```

- /tmp에 /bin/bash를 복사하더라도 의존성때문에 문제가 생긴다. 프로세스는 다른 프로그램이나 파일등에 의존하고 있고 이 경로를 절대 경로로 확인한다

- **ldd** 명령어로 의존 프로그램을 확인한다

```bash
root@s17b59a580ca:/etc/network# ldd /bin/bash
	linux-vdso.so.1 =>  (0x00007ffe988ba000)
	libtinfo.so.5 => /lib/x86_64-linux-gnu/libtinfo.so.5 (0x00007f331f0d1000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f331eecd000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f331eb03000)
	/lib64/ld-linux-x86-64.so.2 (0x000055b65e8aa000)
```

- [가상 공유 파일](https://stackoverflow.com/questions/58657036/where-is-linux-vdso-so-1-present-on-the-file-system)인 linux-vdso.so.1을 제외한 파일을 /tmp 아래로 경로를 유지하면서 이동시켜 준다.
- vDSO
  - [vDSO와 vsyscall](https://junsoolee.gitbook.io/linux-insides-ko/summary/syscall/linux-syscall-3)
  - virtual dynamic shared object(vDSO): 물리적 공간을 차지하지 않는 파일. 매우 작은 가상 라이브러리로 다른 프로세스들이 사용가능하게 한다.
  - 리눅스 시스템콜은 컨텍스트 스위칭 때문에 비용이 크다.
  - vDSO: system call과 달리 kernel space가 아닌 user space에서 이루어지도록 리눅스 커널에서 제공

```
/tmp/
├── bin
│   └── bash
├── lib
│   └── x86_64-linux-gnu
│       ├── libc.so.6
│       ├── libdl.so.2
│       └── libtinfo.so.5
├── lib64
│   └── ld-linux-x86-64.so.2
```

- 이제 chroot가 제대로 실행된다. 루트 디렉토리가 /(호스트에서 /tmp)가 되어있다.

```
root@s17b59a580ca:/etc/network# chroot /tmp /bin/bash
bash-4.3# pwd
/
```

- 격리된 상태에서 파일을 만들어보면 호스트에서 /tmp 밑에 파일이 생성된 것을 볼 수 있다(물론 touch 명령어는 위의 방법으로 복사해야 작동한다)

```
root@s17b59a580ca:/etc/network# chroot /tmp /bin/bash
bash-4.3# touch a
bash-4.3# exit
exit
root@s17b59a580ca:/etc/network# ll /tmp/a
-rw-r--r--  1 root root     0 Sep  7 18:42 a
```

#### cgroups

- 프로세스에서 사용 가능한 CPU, Network, Memory 등의 하드웨어 자원을 그룹 단위로 제어하는 리눅스의 커널 기능
- 여러 장치를 하나의 그룹에서 얼마나 사용할 수 있는지 설정할 수 있다

- system의 cgroup 목록을 조회해보자. cgroup을 조회하는 방법은 파일을 직접보거나 **cgroup-tools**에서 지원하는 명령어(**cgcreate, cgdelete, cgset**등)을 이용한다

```bash
root@s17b59a580ca:~# ps -aux | grep nginx
root      84449  0.0  0.2 124980  9868 pts/0    S+   16:14   0:00 nginx: master process nginx -g daemon off;
root@s17b59a580ca:~# cat /proc/84449/cgroup
11:hugetlb:/
10:perf_event:/
9:freezer:/
8:devices:/user.slice/user-0.slice/session-524.scope
7:pids:/user.slice/user-0.slice/session-524.scope
6:cpu,cpuacct:/user.slice/user-0.slice/session-524.scope
5:memory:/user.slice/user-0.slice/session-524.scope
4:cpuset:/
3:net_cls,net_prio:/
2:blkio:/user.slice/user-0.slice/session-524.scope
1:name=systemd:/user.slice/user-0.slice/session-524.scope
```

- cgroup을 생성해보자

```bash
root@s17b59a580ca:~# cgcreate -a root -g cpu:newcg
root@s17b59a580ca:~# ls /sys/fs/cgroup/cpu/newcg/
cgroup.clone_children  cpuacct.stat   cpuacct.usage_percpu  cpu.cfs_quota_us  cpu.stat           tasks
cgroup.procs           cpuacct.usage  cpu.cfs_period_us     cpu.shares        notify_on_release
```

- **stress** 를 통해서 cpu를 고통받게 하자

```bash
root@s17b59a580ca:~# stress -c 1
stress: info: [92567] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd
root@s17b59a580ca:~# top
%Cpu(s): 50.1 us,  0.2 sy,  0.0 ni, 49.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 92568 root      20   0    7480     88      0 R 100.0  0.0   0:18.36 stress
```

- 새로 만든 cgroup의 cpu를 제한하고, **stress**를 해당 cgroup에서 실행시키면 cpu 사용량이 제한되어 있음을 볼 수 있다.

```bash
root@s17b59a580ca:~# cgset -r cpu.cfs_quota_us=30000 newcg
root@s17b59a580ca:~# cat /sys/fs/cgroup/cpu/newcg/cpu.cfs_quota_us
30000
root@s17b59a580ca:~# sudo cgexec -g cpu:newcg stress -c 1
stress: info: [92723] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd
root@s17b59a580ca:~# top
%Cpu(s): 14.6 us,  0.0 sy,  0.0 ni, 85.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.2 st

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 92724 root      20   0    7480     96      0 R  29.9  0.0   0:04.20 stress
```

- 삭제하자

```bash
root@s17b59a580ca:~# cgdelete cpu:newcg
root@s17b59a580ca:~# ls /sys/fs/cgroup/cpu
cgroup.clone_children  cpuacct.stat          cpu.cfs_period_us  cpu.stat    notify_on_release  tasks
cgroup.procs           cpuacct.usage         cpu.cfs_quota_us   docker      release_agent      user.slice
cgroup.sane_behavior   cpuacct.usage_percpu  cpu.shares         init.scope  system.slice
```


#### linux capabilities

- 프로세스의 권한을 제어하는 기능
- 프로세스는 root로 실행되는 프로세스와 일반 사용자가 실행하는 프로세스로 나뉘며 root 권한으로 실행되는 프로세스를 세부환하는 것이 linux capabilities이다
