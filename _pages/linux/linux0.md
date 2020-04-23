---
layout: post
title:  "우분투 기본"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux0.md
category: linux
---

시작& 종료
===========

shutdown
---------

```console
shutdown [option] 시간 [경고 메시지]
```

- option
  - -r : 시스템 재시작
  - -P : 시스템 강제 종료
  - -h : 시스템 종료
  - -c : 예약된 shutdown 명령 취소
  - +m : 시스템 m분 후 종료
  - -k : 시스템 종료되지는 않고 경고 메시지만 출력(미리 경고 등에 사용)

경고 메시지를 통해 다른 사용자에게 경고

reboot
-----

- option
  - -p : 시스템 종료
  - -f : 강제 재부팅
  - -w : 종료와 재부팅 하지 않고 관련 정보만을 */var/log/wtmp* 에 저장

halt & poweroff & init 0
-------------

- halt : 시스템 즉시 종료
- poweroff : 전원 종료
- init 0 : 시스템 종료

로그아웃
--------

- logout
- exit

- - -

가상 콘솔
=========

- 7개 제공
- Ctrl + Alt + F1 ~ F7
- 여러 명의 사용자가 동시 접속

- - -

Run level
=========

- 시스템 관리를 편하게 하기 위하여 서비스 시행을 단계별로 구분하여 적용하는 것
  - Power Off) : 종료 모드
  - Rescue) : 시스템 복구 모드. 단일 사용자 모드
  - Multi-user) : 사용하지 않음
  - Multi-user) : 텍스트 모드의 다중 사용자 모드
  - Multi-user) : 사용하지 않음
  - Graphical) : 그래픽 모드의 다중 사용자 모드
  - reboot) : 재시작

```sh
ls -l /lib/systemd/system/default.target
```

- 위의 명령어를 통해 현재 컴퓨터의 런레벨을 확인할 수 있음

```sh
ln -sf /lib/systemd/system/multi-user.target /lib/systemd/system/default.target
```

- 위의 명령어를 통해 바꾸면 된다(ln -sf: 심볼릭 링크 파일 생성)


- - -

마운트
=====

- [참고자료1](https://mrrootable.tistory.com/28)
- [참고자료2](https://jhnyang.tistory.com/12)

mount
-------
- 디스크와 같은 물리적인 장치를 리눅스 안의 특정 디렉토리에 연결해주는 것
- 윈도우와 달리 연결시 직접 마운트 해야 함.
- 주로 /media or /mnt에 마운트

- option
  - -a : /etc/fstab 에 있는 파일 시스템을 모두 마운트한다
  - -t : 파일 시스템을 지정. ex) -t ext4
  - -o : 다른 옵션 지정

```sh
//cdrom
mount -t iso9660 /dev/cdrom /media/cdrom
```

- umount : 마운트 해제

fstab
-----

- /etc 밑에 존재하는 시스템 설정 파일.
- 부팅시 자동으로 마운트 되게 설정을 해줄 수 있다.

- 6개의 항
  - filesystem : 장치 이름
  - mount point : 연결될 디렉토리(마운트 포인트)
  - type : 파일 시스템 타입
  - option : rw,auto,noauto 등 지정
  - dump : dump가 필요한지 아닌지
  - pass : 무결성 체크 여부. 0이면 무결성 체크 X. 숫자가 작은 것 부터 무결성 체크 진행.
