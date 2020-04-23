---
layout: post
title:  "process"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_ps.md
---

프로세스
========

- 메모리에서 올라가 있는 프로그램
- Background process: 실행은 되었지만 화면에 표시되지 않는 프로세스.jobs로 확인가능
- Foreground process : 실행되고 화면에 나타나는 프로세스
- PID(프로세스 번호) : 각 프로세스에 할당된 고유 번호. 프로그램이 프로세스가 될 때마다 달라짐
- 작업 번호 : 백그라운드로 실행되는 프로세스의 순서. jobs에 나오는 번호

- - -

ps
==

- [참고자료](https://m.blog.naver.com/PostView.nhn?blogId=kor_secom&logNo=70186675630&proxyReferer=https%3A%2F%2Fwww.google.com%2F)
- 현재 작동하는 프로세스 목록을 출력
- option
  - -e : 모든 프로세스 정보 출력
  - -f : 프로세스의 다양한 정보 출력.UID,PID 포함(풀 포맷)
  - -a : 실행중인 전체 사용자의 모든 프로세스 출력
  - -u : 프로세스를 실행한 사용자 정보와 시작 시간 출력
    - -u 사용자 : 특정 사용자의 프로세스만 출력
  - -x : 제어 터미널을 갖지 않는 프로세스 출력
  - -l : 긴 포맷으로 보여줌
  - -p PID : PID의 프로세스만 보여줌

- 정보
  - USER : 프로세스를 실행시킨 사용자
  - PID : 프로세스 번호
  - %CPU : 최근 1분간 프로세스의 CPU 점유율 퍼센트
  - %MEM : 최근 1분간 프로세스가 사용한 메모리 백분율
  - RSS, VSZ: 현재 프로세스가 사용하는 실제 메모리 크기
  - TTY : 프로세스를 제어하고 있는 터미널
  - STAT : 프로세스 상태 코드. R(실행or대기), S(수면), D(입출력 기다리는 중), T(멈춰 있음), Z(좀비 프로세스), X(완전이 죽은 프로세스)
  - START : 프로세스 시작 시간
  - C : 프로세스 우선 순위
  - COMMAND(CMD) : 프로세스를 생성한 명령어
  - PPID : 부모 프로세스의 PID

![ps]({{stie.baseurl}}/post_img/{{page.name}}/ps.png)

- pstree : 모든 프로세스의 트리 확인

![pstree]({{stie.baseurl}}/post_img/{{page.name}}/pstree.png)

![psgrep]({{stie.baseurl}}/post_img/{{page.name}}/ps_grep.png)

자주 사용하는 방법. **ps -ef|grep 원하는 정보**

- - -

fg, bg, kill, jobs
=====

- [참고자료1](https://se.uzoogom.com/77)
- [참고자료2](https://studymake.tistory.com/621)
- jobs : 작업이 중지된 상태나 백그라운드 프로세스를 보여줌
- 명령어 & : 작업을 백그라운드로 실행

```
//백그라운드로 실행
vim a.tx &

//확인
jobs

//포어그라운드로 변경
fg %JID

//백그라운도 변경
Ctrl+z
bg %JID|PID

//백그라운드 프로세스 일사중지
kill -STOP %JID|PID

//정지된 백그라운드 프로세스 시작
kill -CONT %JID|PID

//종료
kill %JID|PID

//강제종료
kill -KILL %JID|PID
kill -9 %JID|PID
```

![kill]({{stie.baseurl}}/post_img/{{page.name}}/kill.png)

- JID을 사용할 경우 필히 **%** 를 사용해야 한다. 그냥 하면 PID로 인식한다
- ctrl +c : 작업 취소
- ctrl + d : 작업 정상 종료
- ctrl + z : 작업 대기
