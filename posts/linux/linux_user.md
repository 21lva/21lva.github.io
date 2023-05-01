---
layout: post
title:  "사용자, 그룹, 소유권, 허가권"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_user.md
category: linux
---

- 리눅스는 다중 사용자 시스템(multi-user system)
- **root** 라는 super user가 모든 권한을 가지고 있음
- 사용자는 */etc/passwd* 에 정의되어 있음.
- 모든 사용자는 하나 이상의 그룹에 속함

![사용자]({{site.baseurl}}/post_img/{{page.name}}/etc_passwd.png)

- 사용자이름 : 암호 : 사용자 ID : 소속 그룹 ID : 추가 정보 : 홈 디렉토리 : 기본 셸
- 암호는 다른 파일( **/etc/shadow** )에서 관리. 암호화 되어 있다.

- - -

유저, 그룹 CUD
======


- whoami : 나에대한 정보표시
- who : 접속중인 사용자 정보 표시
  - -u : user의 idle time 확인
  - -H : 각 필드의 제목도 표시
  - -a : 모든 정보 표시
  - -b : 시스템 마지막 부팅 정보
  - -r : 런레벨 확인
- w : 서버정보화 사용자 정보를 같이 보여줌
  - 서버의 현재시간
  - 부팅 이후 시스템 작동 시간
  - 접속자 총 수
  - 접속자별 서버 평균부하율
  - 접속자별 계정/ TTY/ IP/ 로그인 시각 /CPU 정보
  - 접속자별 현재 사용 명령어(what)

![w]({{site.baseurl}}/post_img/{{page.name}}/w.png)

- users : 접속중인 사용자의 이름
- adduser : 새로운 사용자 추가. /etc/skel 디렉토리의 파일들을 복사하여 /home 디렉토리에 유저디렉토리 생성.
- passwd : 사용자의 비밀번호를 지정하거나 변경
- usermod : 사용자의 속성을 변경
- userdel : 사용자를 삭제
  - -r : 폴더까지 삭제
- chage : 사용자의 암호를 주기적으로 변경하도록 설정

- adduser

```bash
adduser newuser
adduser --gid 그룹ID newuser
passwd newuser
usermod -g 그룹ID newuser
userdel newuser
userdel -r newuser
chage -m 2 newuser
```

- groups : 현재 사용자가 속한 그룹을 보여줌
- groupadd : 새로운 그룹을 생성
- groupmod : 그룹의 속성을 변경
- groupdel : 그룹삭제
- group

```
groups
groupadd newgroup
groupmod --new-name newgroup2 newgroud
```

- - -

허가권(Permission) & 소유권(Ownership)
=======

![허가권과 소유권]({{site.baseurl}}/post_img/{{page.name}}/po.png)

- 파일타입, 허가권, 링크수(해당 파일이 링크된 수), 소유자, 소유 그룹, 용량, 생성날짜, 파일 이름

허가권
--------

![허가권]({{site.baseurl}}/post_img/{{page.name}}/permission_01.png)

- r(read)w(write)x(execute=실행,접근)의 형태로 나와 있음.

|권한|파일|디렉토리|Octal|
|Read(r)|파일을 읽고 복사 가능|ls를 통해서 내부 확인 가능|4|
|Write(w)|파일의 내용 수정 가능|디렉토리 내의 파일을 수정, 삭제, 추가 가능|2|
|Execute(x)|실행파일을 실행 가능|cd 명령어로 내부로 접근 가능|1|

- file type : 디렉토리, 소켓, 심볼릭 링크, 일반파일 등을 구분
- user : 소유자의 권한
- group : 소유 그룹의 권한
- other : 소유자나 소유그룹에 속하지 않은 사용자의 권한

- chmod : 허가권을 변경.

```
chmod 777 test
```

- test파일의 허가권을 rwxrwxrwx 로 변경
- 각 숫자를 더하면 됨

```
chmod o-x test
chmod ug+w,o=r test
```
  - 다른 사용자들의 실행 권한을 제거
  - 소유자와 소유그룹의 쓰기 권한을 추가, 다른 사용자들의 권한을 read로 설정.

소유권
-----

- 허가권 옆 **newuser ubuntu** 가 **소유자 소유그룹** 이다

- chown : 파일의 소유자 또는 소유자+그룹 변경. super user만 가능
  - -h : 심볼릭 링크 파일 자체의 소유주나 그룹을 변경
  - -R : 하위 디렉토리와 파일들의 소유주도 모두 변경

- chgrp : 파일의 소유 그룹을 변경

```
sudo chown user01 a.out
sudo chgrp grp01 a.out
sudo chown user02.grp02 a.out
```


umask
------

- 파일과 디렉토리를 생성할 때 가지게 되는 기본 허가권을 결정하는 값.
- 최대 허가권(파일:666, 디렉토리:777)- umask = 기본 허가권


- - -

특수권한
=======

setUID
-------

- 사용자가 파일을 실행을 할때, 일시적으로 소유자의 권한을 행사한다

```
chmod 4허가권 파일이름
```

- 기존 허가권에 실행가능(x)가 있었으면 S, 없을 경우에는 s가 x자리에 위치한다.

![setuid]({{site.baseurl}}/post_img/{{page.name}}/setuid.png)

- /etc/passwd에 접근하여 비밀번호를 변경하는 파일인 /usr/bin/passwd는 실행시 일시적으로 root권한을 통해서 /etc/passwd파일을 실행한다.

setGID
-------

- 사용자가 파일을 실행할 때 일시적으로 파일의 소유 그룹의 권한을 가진다

```
chmod 2허가권 파일이름
```

- 기존 허가권에 실행가능(x)가 있었으면 S, 없을 경우에는 s가 x자리에 위치한다.


sticky bit
-------------

- sticky bit가 설정되어 있는 디렉토리에 파일을 생성하면 해당 파일은 생성한 사람이 소유 권한을 가짐.
- 소유자와 root만이 해당 파일에 대한 삭제, 수정 권한 가진다.

![sticky]({{site.baseurl}}/post_img/{{page.name}}/sticky.png)

- 기존 권한에 실행권한이 없으면 T, 있으면 t로 표시
