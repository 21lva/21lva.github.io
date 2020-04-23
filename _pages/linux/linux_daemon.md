---
layout: post
title:  "daemon, cron, at, 서비스, 소켓"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_daemon.md
---


데몬
===

- [참고자료](https://nextline.tistory.com/55)
- 시스템에 관련된 작업을 하는 후위 프로세스(background process)
- 보통 부팅시 시행
- telnet, ftp, http 등이 존재
- 할 일이 없을 때는 idle 상태로 있다가 할 일이 주어지면 시행

실행과 중지
----------

- service 데몬 start
- service 데몬 restart
- service 데몬 stop

standalone & xinetd
--------

#### 1. xinetd
- super daemon에 의해 관리
- 필요한 경우에만 메모리에 적재되어 실행
- xinetd 데몬의 설정 파일은 /etc/inetd.conf에 존재
- /etc/xinetd.d/ 에 존재

```
//설치
sudo apt install xinetd

//시작
/etc/init.d/xinetd start

//재시작
/etc/init.d/xinetd restart

//종료
/etc/init.d/xinetd stop

//상태
/etc/init.d/xinetd status

//설정
vi /etc/xinetd.conf
```

- 설정파일을 열면 다음과 같은 화면이 나온다

![xinetd]({{site.baseurl}}/post_img/{{page.name}}/xinetd.png)

- 아래의 내용을 입력하면 된다

```
# telnet setting
service telnet
{
  disable = no
  flag = REUSE
  socket_type = stream
  wait = no
  user = root
  server = /usr/sbin/in.telnetd
  log_on_failure += USERID
}
```

- option
  - service 서비스이름 : /etc/services 파일에 등록된 이름과 동일해야 함
  - disable : 서비스 사용 하지 않을 여부
  - socket_type = stream  : tcp를 사용한다는 말. stream(TCP), dgram(UDP)
  - server : 서비스 프로그램의 절대 경로
  - server_args : daemon에 넘겨질 인수
  - wait : yes이면 single-tread service이다.
  - group : gid 설정.

- xinetd를 재시작하면 된다.



#### 2. standalone
- 독립적으로 실행.
- 항상 메모리에 상주. 메모리 점유
- 빠른 응답 속도.
- super daemon도 standalone

- - -

cron
=====

- [참고 자료](https://ukzzang.tistory.com/33)
- 일정 시간마다 명령을 자동으로 실행해주는 데몬
- 관련 데몬은 crond
- 관련 파일은 /etc/crontab

- systemctl status cron 으로 동작하는지 확인

![crontab]({{site.baseurl}}/post_img/{{page.name}}/crontab.png)

- 분(m) 시(h) 일(dom) 월(mon) 요일(dow) 사용자(user) 실행명령(command)

- 표현
  - \* : 매 번의 의미
  - , : 여러 시간대 지정
  - \- : 시간을 범위로 지정
  - \\ : 시간의 간격을 지정

- 각 유저의 cron은 **/var/spool/cron/crontabs/사용자** 에 저장된다

crontab 명령어
-------------

- crontab -l 명령어는 각 유저(root포함)가 작성한 것만 보여줌
- /etc/crontab에 저장된 내용은 시스템이 관리하므로 보여주지 않는다

- option
  - -l : 현재 유저의 crontab 내용 출력
  - -e : crontab 내용을 작성하거나 수정
  - -r : crontab 내용을 삭제
  - -u 유저이름 :  root가 해당 사용자의 crontab 다룰 때 사용

![crontab_ubuntu]({{site.baseurl}}/post_img/{{page.name}}/crontab_ubuntu.png)
ubuntu 유저의 crontab을 수정하자

![crontab_all]({{site.baseurl}}/post_img/{{page.name}}/crontab_all.png)
root유저로 접속하여 ubuntu 유저의 crontab을 볼 수 있다. 각 사용자의 cron은 /var/spool/cron/crontabs/ 아래에 저장되어 있다.

사용자 허가 및 거부
-----------------

- /etc/cron.allow : cron 사용을 허가할 유저의 UID 적는다
- vi /etc/cron.deny : cron 사용을 거부할 유저의 UID 작성

- - -

at
===

- 1회성 작업 예약
- at 시간 날짜 입력해서 시작-> 명령어 입력 -> Ctrl+d 로 저장
- at -l : 목록 확인
- atrm job번호 : 예약 제거

![at]({{site.baseurl}}/post_img/{{page.name}}/at.png)

- - -

rrconf
======

- 부팅시 실행될 데몬을 지정

```
//설치
sudo apt install rcconf

//실행
sudo rrconf
```

![rcconf]({{site.baseurl}}/post_img/{{page.name}}/rcconf.png)



- - -

서비스 & 소켓
=====

서비스
-----

- 시스템과 독자적으로 구동되어 제공되는 프로세스
- FTP 서버, DB 서버, 웹서버 등이 존재
- systemctl <start|restart|stop> 서비스이름
- /lib/systemd/system/ 디렉토리에 서비스이름.service의 형태로 스크립트 파일이 존재 -> 이 스크립트를 실행하여 서비스를 실행
- 거의 항상 daemon이다.
- [서비스와 데몬의 차이](https://stackoverflow.com/questions/28139377/daemon-and-service-difference) : 서비스는 다른 프로그램의 요청에 응답하는 프로그램을 말한다. 거의 항상 데몬으로 돌아간다

![cron service]({{site.baseurl}}/post_img/{{page.name}}/cron_service.png)

소켓
----

- 서비스는 항상 가동되지만, 소켓은 외부에서 특정 요청이 들어올 경우에만 systemd가 동작시킨다. 요청이 끝나면 종료
- 서비스보다 응답시간이 길어질 수도 있음
- 관련 스크립트는 /lib/systemd/system 디렉토리에 소켓이름.socket으로 저장되어 있음
- 예전에는 xinetd를 주로 사용하였지만, 이제는 거의 소켓으로 함
- ex) telnet

- 간단하게 다뤘지만, 매우 긴 내용이므로 다음에 더 공부하기로 함
- [더 볼 자료](https://luckyyowu.tistory.com/71)
