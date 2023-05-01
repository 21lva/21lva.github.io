---
layout: post
title:  "Architecture"
date:   2022-06-25 02:02:59
author: Inhyuk
category: linux
tags: linux
cover:  "/assets/instacode.png"
name: linux_os.md
---

리눅스
=====


- 프로그램은 3가지 종류로 구성  
  1. application: 사용자가 직접사용하는 프로그램
  2. 미들웨어: 여러 어플리케이션이 사용하는 프로그램. ex) 데이터베이스
  3. os: 하드웨어에 직접 접근해서 사용하면서 어플리케이션과 미들웨어에 필요한 기능을 제공하는 프로그램


사용자 모드
=======

- os는 커널 외에도 사용자 모드에서 동작하는 프로그램들을 포함.
- 프로그램은 os의 라이브러리를 사용하거나 직접 **시스템 콜**을 호출해서 커널에 하드웨어 조작을 요청


시스템 콜
-------

- 시스템 콜: 프로세스 생성, 하드웨어 조작 등의 기능을 커널에게 요청하는 방법
  1. 프로세스 생성, 삭제
  2. 메모리 확보, 해제
  3. IPC(프로세스간 통신)
  4. 네트워크
  5. 파일시스템
  6. 디바이스 접근

- 시스템 콜은 CPU의 특수한 명령을 실행해야 호출됨. 프로세스는 사용자 모드에서 실행되다가 커널에 시스템 콜을 전달하면 CPU에서 **Interrupt**발생. 인터럽트는 CPU에게 사용자 모드에서 커널모드로 변경을 하게 만든다.
- 커널은 프로세스가 요청한 시스템콜을 수행하기 전에 해당 요구의 유효성을 검증한다.
- 유저프로세스가 시스템 콜 없이 직접 CPU같은 디바이스에 접근하는 방법은 없다.


```
프로세스 ---->시스템 콜            -->동작
---------------|--------------|---
커널            ---->동작------->
---------------|--------------|---
CPU   user mode| kernerl mode | user mode
```

- 리눅스에서 *strace* 명령어로 어떤 시스템콜이 호출 되었는지 확인할 수 있다. 아래의 c명령어를 이용하자

```c
#include <stdio.h>

int main(){
    printf("hello world");
    return 0;
}
```

- compile후에 호출하고 strace를 이용해서 확인해보자.

```
[root@s181376e15b1 ~]# gcc -o hello hello.c
[root@s181376e15b1 ~]# strace -o hello.log ./hello
execve("./hello", ["./hello"], 0x7ffdcb14b4f0 /* 22 vars */) = 0
brk(NULL)                               = 0xc8d000
 |
중략
 |
exit_group(0)                           = ?
+++ exited with 0 +++
```

-  *sar* 명령어를 이용하면 cpu가 어느 mode에서 동작하는지 확인할 수 있다. 모든 CPU가 어떤 영역에서 동작하는지 1초마다 확인해보자.
  - user: 사용자 모드
  - nice: nice로 우선순위를 변경한 프로세스가 사용자 모드에서 CPU를 소비한 시간
  - idle: 대기 상태
  - iowait: cpu가 io wait을 위해 idle 상태로 소비한 시간
  - system: 시스템 모드

- 아래의 프로그램을 컴파일 후 background에서 실행해보자  

```c
#include <sys/types.h>
#include <unistd.h>

int main(){
    for(;;)
         getppid();
    return 0;
}
```

- 그 다음에 sar로 확인하면 다음과 같다.

```
sar -P ALL 1

22시 11분 18초     CPU     %user     %nice   %system   %iowait    %steal     %idle
22시 11분 19초     all     37.13      0.00     20.30      0.50      0.50     41.58
22시 11분 19초       0     14.14      0.00      7.07      0.00      0.00     78.79
22시 11분 19초       1     60.00      0.00     33.00      0.00      0.00      7.00
```

- 1번 cpu가 user와 system을 6:3비율로 변화하는 것을 알 수 있다. 프로그램이 *getppid*를 호출하면 커널모드로 진입하기 때문이다.


wrapper
-------

- 위에서 본 것처럼 시스템 콜은 간접적으로 호출한다.
- 만약 os가 없다면 모든 프로그램은 각각 시스템 콜을 호출할때 마다 아키텍처에 의존적인 어셈블리 언어를 사용하여 호출해야한다. 이러한 방식은 어렵기도 하고 다른 아키텍처에서 사용하기 힘들기 때문에 *system call wrapper*를 사용한다.
- wrapper는 아키텍처별로 존재한다. 모든 프로그램들은 wrapper함수를 호출해서 시스템콜을 간접 요청한다.

표준 c library
----------

- c언어에는 ISO에서 정한 표준 라이브러리가 존재. 보통은 GNU 프로젝트가 제공하는 glibc를 표준 라이브러리로 **link**해서 사용. glibc는 wrapper함수를 포함.

```
[root@s181376e15b1 ~]# ldd hello
	linux-vdso.so.1 =>  (0x00007fffeff79000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f6a68f19000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f6a692e7000)
```

os가 제공하는 프로그램
--------------

- os는 자체적으로 라이브러리 뿐만 아니라 프로그램을 제공한다.
1. init: 시스템 초기화
2. sysctl, nice, sync: os 동작 변경
3. touch, mkdir: 파일
4. grep, sort, uniq: 텍스트 가공
5. sar, iostat: 성능 측정
6. gcc: 컴파일러
7. per, python, ruby: 스크립터 언어 실행 환경
8. bash

- - -

프로세스 관리
=========

- 리눅스에서는 2가지 목적으로 프로세스 생성
  1. 같은 프로그램을 여러 프로세스로 나눠서 처리. fork
  2. 전혀 다른 프로그램을 생성. fork -> execve

fork
----

- 실행한 프로세스(부모 프로세스)와 같은 프로세스(자식 프로세스)가 생성된다. 자식 프로세스의 메모리 영역을 확보하고 부모 프로세스의 메모리를 복사.

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

#include <err.h>


static void child(){
    printf("Child Process\n");
}

static void parent(pid_t pid_c){
    printf("[PARENT PROCESS], child process pid is %d\n",pid_c);
}

int main(void){
    printf("Before fork\n");//fork 전이기 때문에 자식 프로세스는 이 문장을 수행하지 않는다.
    pid_t ret;
    ret = fork();
    if (ret == -1)
        err(EXIT_FAILURE, "fork failed");
    if (ret ==0) {
         //자식 프로세스는 fork 이후부터 수행한다
         //자식 프로세스에서 fork의 반환값은 0이다.
         child();
    } else {
         //부모 프로세스에서 fork의 반환값은 자식 프로세스의 pid이다.
        parent(ret);
    }
    printf("Bye\n");//fork 이후이므로 부모와 자식 모두 이 문장을 수행한다.
    return 0;
}
```

```
[root@s181376e15b1 ~]# gcc -o fork fork.c
[root@s181376e15b1 ~]# ./fork
Before fork
[PARENT PROCESS], child process pid is 18528
Bye
Child Process
Bye
```

execve
--------

- execve는 현재 프로세스의 메모리를 다른 실행파일로 덮어 씌운다. 실행 파일을 읽어서 포르세스의 메모리 맵에 필요한 정보를 읽는다. 이 때 필요한 정보는 코드, 데이터 영역외에도 코드영역과 데이터영역의 오프셋,사이즈, 메모리맵 시작주소, 코드 이외의 변수등에서의 데이터 영역에 대한 같은 정보. 최초로 실행할 명령의 메모리 주소등을 포함한다.

```
|이름                    | 값|
|코드 영역의 파일상 오프셋    |100|
|코드 영역의 사이즈         |100|
|코드 영역의 메모리맵 시작주소 |300|
|데이터 영역의 파일상 오프셋   |200|
|데이터 영역의 사이즈        |200|
|데이터 영역의 메모리맵 시작주소|400|
|엔트리포인트               |300|
|        
|     _________0                
|---->| 보조정보 |                  
      ---------100
      |        ^
      |        |
      |  코드  100
      |        |
      |        v         
실행파일 --------200
      |        ^
      |        |
      |  데이터 200
      |        |
      |        v
      ---------400
          |
          V
      _________0                
      |        |                  
      ---------300(엔트리 포인트)
      |        ^
      |        |
      |  코드  100
      |        |
      |        v         
메모리  --------400
      |        ^
      |        |
      |  데이터  200
      |        |
      |        v
      ---------600
```

- 엔트리 포인트와 보조정보는 다음과 같이 확인가능


```
[root@s181376e15b1 ~]# readelf -h fork
ELF Header:
  ~~생략~~
  Entry point address:               0x400520
  ~~생략~~

  [root@s181376e15b1 ~]# readelf -S fork
  There are 30 section headers, starting at offset 0x19f8:

  Section Headers:
    [Nr] Name              Type             Address           Offset
         Size              EntSize          Flags  Link  Info  Align
  ~~생략~~
    [13] .text             PROGBITS         0000000000400520  00000520
         00000000000001f2  0000000000000000  AX       0     0     16
  ~~생략~~
    [24] .data             PROGBITS         0000000000601048  00001048
         0000000000000004  0000000000000000  WA       0     0     1
  ~~생략~~
```

- 실제로 새로운 프로세스릉 동작시키려면 fork + execve를 수행해야 한다. 다음은 해당 과정을 수행하는 프로그램이다.

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <err.h>

static void child(){
    char *args[] = {"/bin/echo", "hello", NULL};
    printf("Child Process\n");
    fflush(stdout);
    execve("/bin/echo", args, NULL);
}

static void parent(pid_t pid_c){
    printf("[Parent Process] child pid is %d\n",pid_c);
}

int main(){
    pid_t ret;
    ret = fork();
    if (ret == 0){
        child();
    } else {
        parent(ret);
    }
}
```

```
[root@s181376e15b1 ~]# gcc -o fe fe.c
[root@s181376e15b1 ~]# ./fe
[Parent Process] child pid is 21507
Child Process
[root@s181376e15b1 ~]# hello
```

- - -


스케쥴러
======

- 리눅스 커널에는 프로세스 스케쥴러 기능이 존재. 프로세스 여러 가지를 **concurrent**하게 동작시킴.
- 동시에 여러 프로세스를 실행시키더라도 하나의 논리 cpu(하이퍼스레드)에 동작되는 프로세스는 1개이다. 각 프로세스는 Round Robin 방식으로 cpu 자원을 사용.
- 컨텍스트 스위치: 논리 cpu가 수행 중인 프로세스를 바꾸는 과정.

프로세스 상태
--------

- 프로세스는 4가지 상태를 갖는다
  1. 실행 상태(R)
  2. 실행 대기 상태(R)
  3. 슬립 상태(S or D): 이벤트가 발생하기를 기다리고 있는 상태. ex) sleep 호출, io interrupt 발생, 네트워크 interrupt. S는 시그널에 따라 실행 상태로 되돌아 오는 것이고 D는 그렇지 않는 프로세스. D인 프로세스는 보통 수 밀리세컨드 이후로 다른 상태로 바뀐다.
  4. 좀비 상태(Z): 프로세스가 종료한 뒤 부모 프로세스가 종료상태를 인식하기 까지 기다리는 상태

```
[root@s181376e15b1 ~]# ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     4:46 /usr/lib/systemd/systemd --system --deserialize 17
    2 ?        S      0:00 [kthreadd]
    4 ?        S<     0:00 [kworker/0:0H]
    5 ?        S      0:18 [kworker/u30:0]
    ~~~생략~~~
    2817 pts/0    R+     0:00 ps ax
```

- 프로세스의 lifecycle은 다음과 같다

```
프로세스 생성
  \
   -> 실행 대기 상태(Ready)  -----> 실행 상태(Running) -(종료 처리 호출)-> 좀비 상태 -(부모 프로세스가 종료 상태 획득)->프로세스 종료
          ^              <------       |
          |                            |
          ----------슬립 상태----------(interrupt)
```

idle
-----

- 논리 cpu가 아무런 process도 처리하지 않는 상태. 정확히는 *idle process*라는 것을 처리 중이다. 전에 사용한 *sar* 명령어를 이용하여 idle 상태에 있는 프로세스를 확인해보자

```
[root@s181376e15b1 ~]# sar -P ALL 1
Linux 3.10.0-1127.19.1.el7.x86_64 (s181376e15b1) 	2022년 06월 26일 	_x86_64_	(2 CPU)

01시 03분 53초     CPU     %user     %nice   %system   %iowait    %steal     %idle
01시 03분 54초     all      4.00      0.00      2.00      0.50      0.00     93.50
01시 03분 54초       0      0.00      0.00      2.00      0.00      0.00     98.00
01시 03분 54초       1      6.93      0.00      2.97      0.99      0.00     89.11
```

- 아래는 사용자의 입력을 받아서 처리하는 동작을 반복하는 프로세스와 해당 논리 CPU의 상태를 대략적으로 그린 모습이다

```
동작|     실행    입력대기    입력    파일읽기    파일읽기종료  프로세스종료
---------V------v---------v------v---------v--------v--------
프로세스0| |실행상태|슬립상태    |실행상태|슬립 상태  |실생상태  |
-------------------------------------------------------------
논리cpu|  |p0    |idle     |p0    |idle     |p0      |idle
```

throughput, latency
------------------

- throughput: 단위 시간당 처리된 일의 양. #(완료한프로세스)/경과 시간. 논리 cpu의 연산 리소를 최대한 사용할수록(=idle이 적을수록) 좋음. 논리 cpu의 idle상태가 없다면 프로세스의 수를 늘려도 throughput은 변함이 없다.
- latency: 프로세스가 시작부터 종료까지 경과된 시간. 종료시간 - 시작시간. 프로세스의 수를 늘리면 각 프로세스의 레이턴시는 악화된다.
- throughput과 latency는 trade-off관계: 프로세스의 수가 늘어나면 idle의 상태가 적어서 throughput은 증가하지만 각 프로세스의 레이턴시는 길어진다.

복수의 논리 cpu
-------------

- 논리 cpu가 복수인 경우에는 프로세스를 어떤 논리 cpu에게 할당해야 하는지를 결정하는 *load balancer(global scheduler)*가 있다.

경과 시간 & 사용 시간
------------------

- 경과 시간: 프로세스가 시작해서 종료할 때까지의 시간.
- 사용 시간: 프로세스가 cpu에서 동작하는 시간. 경과시간 = 사용시간 + 대기 시간 + 슬립 시간 + alpha
- *time* 명령어를 이용하면 프로세스의 사용시간을 측정할 수 있다. 먼저 1개의 논리 cpu에 1개의 프로세스를 동작시켜보자. *taskset -c n <실행명령어>*: 논리 cpu n에서 프로세스 수행
```
[root@s181376e15b1 ~]# time taskset -c 0 ./loop

real	0m14.386s      => 경과 시간
user	0m14.359s      => 사용 시간 중 user mode에서의 시간
sys	0m0.001s         => 사용 시간 중 kernel mode에서의 시간

[root@s181376e15b1 ~]# sar -P 0 1
Linux 3.10.0-1127.19.1.el7.x86_64 (s181376e15b1) 	2022년 06월 26일 	_x86_64_	(2 CPU)

01시 32분 43초     CPU     %user     %nice   %system   %iowait    %steal     %idle
01시 32분 44초       0    100.00      0.00      0.00      0.00      0.00      0.00
```


- 같은 명령어를 2개의 논리 cpu에서 수행시켜보면 하나의 cpu에서만 해당 프로세스를 담당하는 것을 알 수 있다.

```
[root@s181376e15b1 ~]# taskset -c 0,1 ./loop

real	0m37.980s
user	0m37.832s
sys	0m0.003s

[root@s181376e15b1 ~]# sar -P ALL 1
Linux 3.10.0-1127.19.1.el7.x86_64 (s181376e15b1) 	2022년 06월 26일 	_x86_64_	(2 CPU)

01시 33분 44초     CPU     %user     %nice   %system   %iowait    %steal     %idle
01시 33분 45초     all     59.30      0.00      2.51      0.00      0.00     38.19
01시 33분 45초       0    100.00      0.00      0.00      0.00      0.00      0.00
01시 33분 45초       1     18.37      0.00      4.08      0.00      0.00     77.55
```

- 이번에는 시작하자마자 슬립 상태로 들어갔다가 일정 시간이 지난 후에 깨어나서 바로 종료되는 프로세스를 동작시켜보자

```
[root@s181376e15b1 ~]# time taskset -c 0 sleep 10

real	0m10.002s    
user	0m0.000s      => 슬립상태로 들어가기 때문에 사용시간은 많지 않다.
sys	0m0.002s
```

- time 외에도 *ps -eo* 명령어로 *etime*(경과시간), *time*(사용시간)등을 측정할 수 있다.

```
[root@s181376e15b1 ~]# ps -eo pid,comm,etime,time
  PID COMMAND             ELAPSED     TIME
  10434 airflow         19-05:47:39 08:29:40   => 프로세스 생성일로부터 19일 5시간 47분 39초 지났고, cpu 사용시간은 8시간 29분 40초이다
```

우선 순위 변경
------------

- *nice*는 프로세스의 우선순위를 -19부터 20까지 설정. 기본값은 0이며 작을수록 우선순위가 높다. 우선순위를 낮추는 것(숫자를 증가시키는 것)은 누구나 가능하지만 반대의 과정은 root 권한이 있을 때만 가능하다.

- - -

메모리
=====

- 리눅스는 커널의 메모리 관리 시스템으로 프로그램이 사용하는 메모리와 커널 자체가 사용하는 메모리를 관리.
- 메모리 정보는 *free*명령어와 *sar -r*로 확인 가능

```
[root@s181376e15b1 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.2G        216M        192M        2.3G        2.0G
Swap:            0B          0B          0B

[root@s181376e15b1 ~]# sar -r

00시 00분 01초 kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
00시 10분 01초    247144   3623336     93.61      2000   2161224   1840568     47.55   2088384   1053312       172
```

- free에서 나오는 정보
  1. total: 전체 메모리
  2. used: 사용중인 메모리
  3. free: 표기상 이용하지 않는 메모리
  4. avaliable: 실질적으로 사용 가능한 메모리. free 값이 부족하면 해제되는 커널내의 메모리 영역(버퍼 캐시나 페이지 캐시의 대부분 + 다른 커널내의 메모리 일부)을 포함. available = free + 해제가능 메모리
  5. buff/cache: 버퍼 캐시 또는 페이지가 이용하는 메모리.

- sar에서 나오는 정보
  1. kbmemfree: free의 free 정보와 같음
  2. kbbuffers: buff 메모리양
  3. kbcached: cache 메모리 양. kbbuffers + kbcached  = buff/cache

```
<------------ total ---------------------->
|프로세스용 메모리 |free|   커널의 메모리          |
|              |    |buff/cache|           |
|              |    |해제불가| 해제가능 |해제불가 |
```

- free의 메모리가 부족해지면 커널은 available영역을 해제.(한 번에 해제하지는 않음) -> available 영역도 줄어서 해제할 영역이 줄어들면 **OOM** 에러 발생 -> 커널의 **OOM Killer**가 적절한 프로세스를 선택해서 강제로 kill. kill 대상이 되고 싶지 안으면 systctldㅢ *vm.panic_on_oom*의 값을 0(기본값)에서 1로 변경하면 메모리 부족시 kill이 일어나지 않고 시스템을 종료시킨다.

가상 메모리가 없을 때의 메모리 할당
---------

- 커너리 메모리를 할당하는 경우: 1) 프로세스를 생성 2) 프로세스를 생성한 후 추가로 동적으로 메모리가 필요한 경우
- 메모리에 직접 접근 가능하게 하면 보통 프로세스가 추가로 메모리가 필요하면 커널에 메모리 확보용 시스템 콜 호출 -> 커널은 필요한 사이즈의 빈 메모리 영역을 고정하고 그 시작 주소값을 반환. 이러한 프로세스들이 메모리에 직접 접근하는 방식에는 3개의 문제 존재한다.
  1) 메모리 단편화
  2) 할당되지 않은 메모리에 접근 가능: 시작 주소값이 아닌 다른 메모리에 직접 접근 가능
  3) 여러 프로세스를 다루기 곤란: 예를 들어, 두 프로그램을 실행할 때 각자의 실행 파일에 있는 보조정보를 메모리에 옮긴다. 그러나 두 보조정보의 코드 영역의 파일상 오프셋 값이 같은 값을 가지면 두 정보가 겹치면서 문제가 발생한다.

Virtual memory
----------

- 가상 메모리는 시스템의 메모리의 물리적 주소 직접 접근하지 않고 가상 주소(논리적 주소)를 사용하여 간접 접근하는 방식.
- *readelf*와 */proc/{pid}/maps*의 메모리 주소는 가상 주소이다.

```                    
                  물리 메모리
                    |   |  <- 0
                    |   |
                    |   |
0 -> | | ---------->|   |  <- 400
|    | |            |   |
가상  | |            |   |   
주소  | |            |   |
공간  | |            |   |
|    | |            |   |
300->| | ---------->|   |  <- 700
                    |   |
                    |   |
                    |   |  <- 1000
```

#### page table

- 페이지: 가상 메모리를 일정한 단위(x86_64의 경우에는 4KB)로 관리하기 위헤 잘라 놓은 단위. 이에 대응하는 물리 메모리는 *frame*.
- 페이지 테이블: 페이지의 가상 주소를 물리 주소로 변환할 때 사용하는 테이블.
- page fault: 물리 주소가 할당되지 않은 가상 주소에 접근하는 경우에 발생. SWAP을 쓰지 않는 경우: 커널의 *page fault handler* 가 **SISSEGV** 시그널을 프로세스에 전달. 프로세스는 강제 종료됨.

#### 프로세스에 메모리 할당

- 프로세스를 생성하는 경우: 커널은 파일의 보조정보를 읽고 필요한 코드 영역 사이즈와 데이터 영역 사이즈를 확인 -> 물리 메모리에 파일의 코드, 데이터 영역을 복사 -> 페이지 테이블에 업데이트(가상주소-물리주소 pair)
- 동적으로 메모리가 필요한 경우: 커널은 메모리의 빈 공간을 고정 -> 페이지 테이블 업데이트

#### 고수준 레벨

- C언어의 경우 표준 라이브러리(glibc)의 *malloc*을 이용하여 메모리 확보. 리눅스에서는 glibc에서 *mmap*이라는 시스템 콜을 이용하여 메모리 확보.
- malloc은 byte 단위로 메모리를 확보하지만 glibc은 mmap을 이용하여 페이지 단위(4KB)로 할당 받아서 메모리 풀 생성. malloc이 호출될 때 필요한 만큼 메모리 풀에서 바이트 단위로 메모리 공간 전달. 이 때 메모리가 부족하면 다시 페이지 단위로 mmap으로 메모리 확보
- 파이썬의 경우에는 C언어의 malloc을 이용하여 메모리 확보.

#### 메모리 단편화(외부 단편화) 문제 해결

- 프로세스의 페이지 테이블을 설정하면 논리 주소와 실제 주소를 다르게 가져갈 수 있다.
- 페이지를 잘게 잘라놓고 필요한 만큼 할당하기 때문에 메모리의 외부 단편화 문제는 방지할 수 있다.

#### 다른 용도의 메모리에 접근하는 문제 해결

- 모든 가상 주소 공간과 페이지 테이블은 프로세스 마다 생성된다
- 어떤 프로세스는 자신의 가상 주소 공간의 메모리에만 접근 가능하고 그 외에는 접근 불가능. 다른 프로세스의 메모리에는 접근 불가능
- 커널 자체가 사용하는 메모리에 대응하는 페이지 테이블의 엔트리는 CPU가 커널 모드로 실행될때만 접근 가능한 *커널 모드 전용*이라는 정보가 추가됨 -> 사용자 모드에서는 접근 불가능.
- 커널의 메모리가 프로세스의 가상 주소 공간에 매핑된 이유
  1.


가상 메모리 응용
-------------

- 가상메모리는 기초적인 것 외에도 여러 방식으로 사용 됨

#### file map

- 프로세스가 파일에 접근할 때는 *read*, *write*, *lseek* 등의 시스템 콜 사용
- *mmap*을 이용하면 파일의 내용을 메모리에 읽어 들여서 가상 주소 공간에 매핑 가능.
- 접근한 영역에 write하게 되면 특정한 타이밍(즉시가 아님)에 해당 정보를 보조 메모리(hdd, ssd 등)에 저장

#### demand paging

- 디멘드 페이징: 프로세스가 가상 주소 공간의 특정 주소에 처음 접근할 때 그 주소에 대응되는 물리 메모리가 할당되는 방식. 할당 받아놓고 안쓰는 메모리가 생기는 문제를 방지.
- 프로세스가 생성할때나 동적으로 메모리 공간을 할당 받을 때 페이지에 *가상 공간 확보 받음*이라고 작성하고 메모리는 할당하지 않음 -> cpu가 페이지 테이블을 이용하여 실제 메모리 주소에 접근하려 함 -> page fault -> page fault handler가 물리 메모리 할당 -> cpu가 기존 동작 수행
- 가상 메모리 부족: 프로세스가 가상 주소 공간의 범위를 넘도록 가상 메모리를 요청하는 경우에 발생. x86의 경우 주소를 표현하는데에 4Byte(=8*4bit=32bit)이기 때문에 가상 공간이 4GB로 한정된다. 이 범위를 넘어서는 가상 메모리 공간을 확보하길 원하면 문제 발생. x86_64의 경우에는 8byte를 사용하고 있기 때문에 거의 발생하지 않음
- 물리 메모리 부족: 물리적인 메모리가 부족해지면 발생. 디맨드 페이징을 통해 미리 가상 할당 받은 메모리를 실제로 사용할 때 물리 메모리를 할당 받는 데 이 때 둘의 시간 차이에 의해 발생할 수 있음

#### Copy On Write

- *fork*로 프로세스를 생성할 때는 부모의 메모리를 즉시 복사하는 것이 아니라 부모의 페이지 테이블을 복사한다. 이 때 각 테이블은 쓰기 권한을 제공하지 않는다.

```
<부모 프로세스의 페이지 테이블 >
----------------------------        
|가상주소 | 물리 주소 | 쓰기 권한 |        
|-------|---------|---------
|0~100  | 200~300 | x      |
|-------|---------|--------|
|100~200| 300~400 | x      |
----------------------------

<자식 프로세스의 페이지 테이블 >
----------------------------
|가상주소 | 물리 주소 | 쓰기 권한 |
|-------|---------|---------
|0~100  | 200~300 | x      |
|-------|---------|--------|
|100~200| 300~400 | x      |
----------------------------

물리 메모리
|       | 0~100
|       | 100~200
|부모 자식| 200~300
|공유메모리| 300~400
|       | 400~500
|       | 500~600
|       | 600~700
```

- 부모 자식 중 어떤 프로세스가 메모리에 쓰기 작업을 수행하면 쓰기 권한이 없기 때문에 CPU에서 page fault 발생 -> CPU가 커널 모드로 변경되어서 page fault handler 작동 -> handler가 접근한 페이지를 다른 장소에 복사후 쓰기 작업을 수행한 프로세스에 해당 페이지 할당 -> 부모와 자식 프로세스 모두에 대해 페이지 테이블 entry를 업데이트


```
<부모 프로세스의 페이지 테이블 >
----------------------------        
|가상주소 | 물리 주소 | 쓰기 권한 |        
|-------|---------|---------
|0~100  | 200~300 | x      |
|-------|---------|--------|
|100~200| 300~400 | O      |
----------------------------

<자식 프로세스의 페이지 테이블 >
----------------------------
|가상주소 | 물리 주소 | 쓰기 권한 |
|-------|---------|---------
|0~100  | 200~300 | x      |
|-------|---------|--------|
|100~200| 400~500 | O      |
----------------------------

물리 메모리
|       | 0~100
|       | 100~200
|공유메모리| 200~300
|부모용   | 300~400
|자식용   | 400~500
|       | 500~600
|       | 600~700
```

#### SWAP

- 물리 메모리가 부족하면 **OOM**이 발생. 리눅스에서는 SWAP을 이용하여 보조 메모리를 실제 물리 메모리인 척 사용 가능. 물리 메모리가 부족한 상태에서 프로세스가 물리 메모리를 획득하고자 한다면 기존에 사용하면 물리 메모리의 일부를 보조메모리(SSD, HDD의 SWAP 영역)에 저장하고 그 빈 메모리 공간을 할당.

아래의 그림은 현재 메모리가 모두 사용중인 상태이다.

```
<부모 프로세스의 페이지 테이블 >
------------------        
|가상주소 | 물리 주소 |      
|-------|---------|
|0~100  | 200~300 |
|-------|---------|
|100~200| 300~400 |
|-------|---------|
|200~300|         |
-------------------

물리 메모리
|사용중    | 0~100
|사용중    | 100~200
|프로세스1용| 200~300
|프로세스1용| 300~400
|사용중    | 400~500
|사용중    | 500~600
|사용중    | 600~700
```

이 때 프로세스가 200~300의 가상 메모리 공간에 물리적 공간이 필요하다고 느끼면 OOM이 발생한다. SWAP영역이 설정되어 있으면 물리 메모리를 SWAP 영역으로 옮긴다. 이를 **swap out**(page=out)라고 한다.
그 후에 빈 프레임을 할당한다(실제로는 swap 관리용 장소에도 기록)

```
<부모 프로세스의 페이지 테이블 >
------------------        
|가상주소 | 물리 주소 |      
|-------|-----------|
|0~100  | 200~300   |
|-------|-----------|
|100~200|SWAP의 0~100|
|-------|-----------|
|200~300|  300~400  |
---------------------

물리 메모리                 SWAP
|사용중    | 0~100      |100~200 임시저장|0~100
|사용중    | 100~200    |              |100~200
|프로세스1용| 200~300    |              |200~300
|비어짐    | 300~400    |              |300~400
|사용중    | 400~500
|사용중    | 500~600
|사용중    | 600~700
```

추후에 가상 공간 100~200의 정보가 필요하면 CPU에서 page fault가 발생하고, page fault handler는 **swap in**(=page in)을 수행한다.

- swapping(swap in + swap out)은 물리 메모리를 더 크게 보인다는 장점이 있지만, 느린 보조 메모리에 접근을 하기 때문에 성능에 영향을 미친다. page fault와 스와핑이 지속적으로 발생하는 현상을 **thrashing**이라 하는데 이는 성능을 악화시키는 요인이 된다. 특히 프로세스의 수를 무작정 늘리기만 하면 발생할 수 있다.

- swap영역을 생성해보자. 스왑 영역을 생성하는 방식은 스왑 파티션 생성하는 방식과 스왑 파일을 생성하는 방식이 있는데 여기서는 후자로 만들어본다.
- 지금 현재는 스왑 영역이 설정되어 있지 않다.

```
[root@s181376e15b1 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3779        1210         210         184        2359        2105
Swap:             0           0           0
```

- 현재 나의 보조 메모리 정보는 다음과 같다.

```
[root@s181376e15b1 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
~~생략~~~
/dev/xvda1       50G  6.7G   44G  14% /
~~생략~~~
```

- swap으로 쓰일 file을 생성하자. 생성 후에 시스템에서만 접근 가능하게 권한을 변경한다. 내용을 추가로 설명하자면
  - dd: 블록 단위로 파일을 복사하는 명령어
  - if=File: 복사 대상이 되는 파일. 여기서 사용한 /dev/zero는 초기화 용도로 사용
  - of=File: 복사 파일이 저장되는 위치
  - bs=xx: 블록의 크기 지정
  - count=xxx: bs에 지정한 블록의 갯수

```
[root@s181376e15b1 /]# dd if=/dev/zero of=/swapfile count=2048 bs=1M
2048+0 records in
2048+0 records out
2147483648 bytes (2.1 GB) copied, 9.93966 s, 216 MB/s
[root@s181376e15b1 /]# chmod 600 /swapfile
[root@s181376e15b1 /]# ll -h /swapfile
-rw------- 1 root root 1.9G  6월 27 17:08 /swapfile
```

- 이제 swap 파일을 등록해보자

```
[root@s181376e15b1 /]# mkswap /swapfile
Setting up swapspace version 1, size = 2097148 KiB
no label, UUID=d55ef657-9f59-47b4-a9c2-3332bbb91d68
[root@s181376e15b1 /]# swapon /swapfile
[root@s181376e15b1 /]# swapon -s
Filename				Type		Size	Used	Priority
/swapfile                              	file	2097148	0	-2
[root@s181376e15b1 /]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.1G        212M        184M        2.4G        2.1G
Swap:          2.0G          0B        2.0G
```

사용하던 swap파일은 다음과 같이 삭제한다

```
[root@s181376e15b1 /]# swapoff /swapfile
[root@s181376e15b1 /]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.1G        213M        184M        2.4G        2.1G
Swap:            0B          0B          0B
[root@s181376e15b1 /]# rm /swapfile
```

#### 계층형 페이지 테이블

- 이제 까지 설명할 때 사용한 페이지 테이블은 정말 간단한 페이지 테이블이다.
- x86_64를 기준으로 가상 주소 공간의 최대 크기는 128TB, 페이지의 크기가 4KB, 페이지 테이블의 entry(row)의 크기는 8Byte => 프로세스당 페이지 테이블의 크기 = entry의 크기 * 페이지의 개수(주소공간/페이지의 크기) = 8Byte * (128TB/4KB) = 256GB >>> 보통 PC의 메모리 크기
- 페이지 테이블을 계층형으로 나누면서 메모리를 절약할 수 있다.

- 페이지 테이블이 생성될 때 아래와 같이 1차 테이블을 갖고 시작한다

```
<페이지 테이블 >
-------------------
|가상주소 |         |
|-------|---------|
|0~200  |         |
|-------|---------|
|200~400|         |
|-------|---------|
|400~600|         |
-------------------
```

- 이때 0~200 메모리 주소에 접근한다면 2차 페이지 테이블이 생성된다


```
<페이지 테이블 >
-------------------
|가상주소 |         |
|-------|---------|
|0~200  |  1번    |
|-------|---------|
|200~400|         |
|-------|---------|
|400~600|         |
-------------------

<2차 페이지 테이블 1번>
------------------
|가상주소 | 물리 주소 |
|-------|---------|
|0~100  | 200~300 |
|-------|---------|
|100~200| 400~500 |
-------------------
```

- 추후에 200~400사이의 메모리 주소에 접근을 하게 되면 새로운 2차 페이지 테이블 생성


```
<페이지 테이블 >
-------------------
|가상주소 |         |
|-------|---------|
|0~200  |  1번    |
|-------|---------|
|200~400|  2번     |
|-------|---------|
|400~600|         |
-------------------

<2차 페이지 테이블 1번>
------------------
|가상주소 | 물리 주소 |
|-------|---------|
|0~100  | 200~300 |
|-------|---------|
|100~200| 400~500 |
-------------------

<2차 페이지 테이블 2번>
------------------
|가상주소 | 물리 주소 |
|-------|---------|
|200~300| 500~600 |
|-------|---------|
|300~400| 600~700 |
-------------------
```

- 이 방식은 기존의 테이블 방식보다 초기 페이지 테이블이 작은 효과가 있지만 메모리에 계속 접근하다보면 기존 방식보다 페이지 테이블이 더 커지게 된다. 물론 보통 이런 경우는 많이 없다.
- 시스템이 겪은 메모리 부족 현상 중에는 프로세스를 너무 많이 생성하거나 가상 메모리를 많이 사용하는 프로세스를 생성해서 페이지 테이블의 영역이 커지는 것이 원인으로 발생하는 경우들이 존재한다. 전자는 프로세스의 양을 줄이는 방법으로 해결하고 후자는 **Huge page**방식으로 해결할 수 있다.

#### Huge Page

- 가상 메모리를 대량으로 사용하는 프로세스는 페이지 테이블이 커질 수 밖에 없고 이 페이지 테이블이 차지하는 메모리 양도 늘어나게 된다. 이러한 프로세스를 *fork*하는 경우에 매우 느리다. fork는 페이지 테이블에 대해서는 CoW를 하지 않고 새로 생성하기 때문이다. Huge page를 이용하면 페이지 테이블에 사용되는 메모리 양을 줄일 수 있다.
- 아래는 간단한 Huge page의 페이지 테이블이다. 기존보다 1/4 정도 테이블의 엔트리가 줄어든 것을 알 수 있다. 단점으로는 단편화(특히 내부)가 발생할 수 있다는 것이다.

```
<간단한 Huge page의 페이지 테이블 >
--------------------
|가상주소  |  물리 주소|
|--------|---------|
|0~400   | 100~200 |
|--------|---------|
|400~800 | 200~300 |
|--------|---------|
|800~1600| 300~400 |
--------------------
```

- Transparent Huge page: 가상 주소 공간의 연속된 page가 특정 조건을 만족하면 이 page들을 묶어서 Huge page로 만들어주는 기능. 합치는 작업과 특정 조건이 깨졌을 때의 분해하는 작업등이 성능의 오버헤드가 될 수 있다. 이 기능은 켜고 끌 수 있다. 지금 현재 나의 리눅스는 Huge page가 항상 되도록 되어있다.

```
[root@s181376e15b1 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
```

- - -

계층형 메모리
============

- 메모리는 여러 계층으로 됨. 레지스트->캐쉬->메인메모리->보조메모리(저장장치)

cache
------

- 레지스터 안에서 계산한 결과와 메모리에서 가져온 데이터를 임시로 저장해놓는 메모리. 메모리에서 레지스터로 데이터를 읽어올 때 캐쉬에 *cache line size*만큼 메모리에서 캐쉬로 읽어들인다.
- cache의 장점은 **국소성(locality)**에 의해 얻어짐.
  - 시간 국소성: 어떤 데이터를 접근한 후에 근미래에 접근할 확률이 높음
  - 공간 국소성: 어떤 데이터는 그 주변의 데이터와 같이 쓰일 확률이 높음
- 보통은 cpu내에 내장
- 쓰기 과정
  1. cpu가 레지스터에 데이터 저장
  2. 메모리에서 캐쉬에 데이터 저장 후 변경이 있는 라인에 *dirty* flag를 설정한다
  3. 백그라운드로 메인 메모리에 해당 라인을 저장(write back)한 후 더티 플래그 삭제
- 캐쉬 메모리에 자리가 없는 경우에는 라인 하나를 파기한다. 이 때 해당 라인이 dirty 상태이면 write back을 한다. 캐쉬 라인의 자리가 부족하고 모든 캐쉬 라인이 dirty 상태이면 캐쉬 메모리에 접근할 때마다 데이터 이동이 자주 발생하는 **thrashing** 현상 발생.

- 캐쉬 메모리는 계층형으로 되어 있다: L1, L2, L3. 캐쉬 정보는 */sys/devices/system/cpu/cpu{cpu번호}/cache/index{캐쉬번호}* 를 확인하면 알 수 있다. 아래는 0번 cpu의 L1 캐쉬의 정보이다
  - type: cache 데이터의 종류. Data=데이터만, Code=코드만, Unified=전체
  - shared_cpu_list: 캐쉬를 공유할 **논리 CPU**
  - size: 파일 사이즈
  - coherency_line_size: cache line size

```
[root@s181376e15b1 ~]# ls /sys/devices/system/cpu/cpu0/cache/index0/
coherency_line_size  id  level  number_of_sets  physical_line_partition  shared_cpu_list  shared_cpu_map  size  type
[root@s181376e15b1 index0]# cat /sys/devices/system/cpu/cpu0/cache/index0/type
Data
[root@s181376e15b1 index0]# cat /sys/devices/system/cpu/cpu0/cache/index0/shared_cpu_list      
0
[root@s181376e15b1 index0]# cat /sys/devices/system/cpu/cpu0/cache/index0/size
32K
[root@s181376e15b1 index0]# cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size
64


//나의 컴퓨터는 cpu1개. core 2개. 하이퍼쓰레드가 안되어 있어서 논리적 코어는 2개이다.
[root@s181376e15b1 index0]# grep "physical id" /proc/cpuinfo | sort -u | wc -l
1
[root@s181376e15b1 index0]# cat /proc/cpuinfo | grep processor | wc -l
2
[root@s181376e15b1 index0]# cat /proc/cpuinfo | egrep 'siblings|cpu cores' | head -2
siblings	: 2
cpu cores	: 2
[root@s181376e15b1 index0]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
```

TLB
----

- 캐시 메모리에 있는 데이터를 위해서 메인 메모리에 존재하는 페이지 테이블에 계속 접근 하는 것은 오버헤드가 발생할 수 있음. Translation lookaside buffer는 캐쉬 메모리에서 페이지 테이블과 같은 역할을 하는 장소.

Page cache
----------

- 저장 장치 내의 파일 데이터를 메모리에 캐싱한 것.
- 프로세스가 파일의 데이터를 읽어올 때 커널이 프로세스의 메모리에 파일 데이터를 직접 복사하지는 않고, 먼저 커널 메모리에 있는 페이지 캐시라는 영역에 복사하고 그 후에 프로세스에 복사.
- 읽기로 캐싱한 파일 데이터는 다른 파일에서도 접근 가능. 읽기 시에 (파일이름, 파일 오프셋, 메모리 주소, 더티 유무)의 정보를 커널의 특정 영역에 저장.
- 쓰기시에 프로세스는 페이지 캐시에 데이터를 저장. 커널은 해당 (파일 이름, 파일 오프셋, 메모리 주소, 더티 유무)의 더티를 true로 만든다. 이렇게 쓰였지만 저장이 안된 페이지를 **dirty page**라고 부른다. dirty page는 커널의 백그라운드로 실제 파일에 저장된다.
- 캐쉬 영역이 부족하면 커널은 먼저 더티가 아닌 페이지를 파기해서 자리를 마련한다. 그래도 부족하다면 write back 후에 페이지를 파기한다. 동기환 파일 쓰기를 원한다면 open시에 **O_SYNC**를 설정하면 된다.

- 버퍼 캐시: 페이지 캐시랑 비슷하지만 여기는 파일이 아닌 디바이스 데이터를 캐싱해놓는다.

튜닝
---

- 리눅스에서 *vm.dirty_writeback_centisecs*를 수정하면 write back이 발생하는 주기를 수정할 수 있다. 단위늰 0.01 초.

```
[root@s181376e15b1 index0]# sysctl vm.dirty_writeback_centisecs
vm.dirty_writeback_centisecs = 500
```

- *vm.dirty_background_ratio*: 더티 페이지가 이 비율 만큼 차지하면 백그라운드 write back 처리가 동작한다
- *vm.dirty_background_bytes*: 더티 페이지가 이 byte 만큼 차지하면 백그라운드 write back 처리가 동작한다

```
[root@s181376e15b1 index0]# sysctl vm.dirty_background_ratio
vm.dirty_background_ratio = 10
[root@s181376e15b1 index0]# sysctl vm.dirty_background_bytes
vm.dirty_background_bytes = 0         ===> 설정 안되어 있음
```

- *vm.dirty_ratio*: 지정된 퍼센트를 초과하면 프로세스에 의한 파일에 쓰기가 동기적으로 수행됨(write back 이 즉시 일어남)

```
[root@s181376e15b1 index0]# sysctl vm.dirty_ratio
vm.dirty_ratio = 30
```

- 캐쉬 데이터를 비워보자

```
[root@s181376e15b1 index0]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.1G        1.1G        184M        1.4G        2.1G
Swap:            0B          0B          0B
[root@s181376e15b1 index0]# echo 3 > /proc/sys/vm/drop_caches
[root@s181376e15b1 index0]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        1.0G        2.4G        184M        252M        2.3G
Swap:            0B          0B          0B
```

- - -

파일 시스템
==========

- 리눅스에서는 저장 장치에 직접 접근하지 않고 **File system**을 이용하여 접근
- 파일시스템: 어느 데이터가 어디에 있고, 빈 영역이 어딘지 등을 관리하는 시스템. 데이터를 이름, 위치, 사이즈 등의 보조 정보를 추가하여 파일 단위로 관리. ext4, XFS, Btrfs등이 존재
- 리눅스에서는 카테고리별로 **Directory**를 만들어서 파일 및 다른 디렉토리를 보관. 디렉토리들은 Tree 구조로 되어 있음.
- 리눅스에서는 여러 파일 시스템을 다룬다. 각각의 저장장치는 구조와 파일 사이즈, 파일 시스템의 사이즈, 처리 속도 등이 다르다. 그래도 사용자는 단순히 통일된 시스템 콜을 사용해서 사용 가능.
  - 프로세스의 시스템 콜 -> 커널의 파일 공통처리 시스템에서 파일 시스템에 맞는 프로세스 호출 -> 데이터를 처리하는 디바이스 드라이버(리눅스에 하나 존재)에 의뢰 -> 디바이스드라이버의 작업

```
                       프로세스
                         | <- 시스템 콜
                         v
    -----------------------------------------
    |                    |                  |
    V                    V                  V
ext4처리용 프로세스  XFS처리용 프로세스  Btrfs처리용 프로세스
    |                    |                  |
    V                    V                  V
    -----------------------------------------
                 디바이스 드라이버
    ------------------------------------------
    |                    |                  |
    V                    V                  V
   저장장치A              저장장치B           저장장치C
```

Data & Metadata
---------------

- 파일 시스템에는 데이터(사용자가 작성한 데이터)와 메타데이터(파일 이름, 위치, 종류, 시간 및 권한 정보)가 존재. 메타데이터는 파일에 비해 크지는 않지만, 작은 파일을 많이 만든다면 메타데이터가 차지한 용량 때문에 문제가 발생할 수 있음. *df*로 얻어지는 정보에는 메타데이터 사이즈도 포함.

Quota
-----

- 쿼터: 특정 용도가 파일 시스템의 용량의 대부분을 차지해서 문제가 생기는 것을 방지하기 위한 기능
  1. 사용자 쿼터: 사용자별로 용량을 제한하는 기능. ext4, XFS에서 사용 가능.
  2. 디렉토리 쿼터: 특정 디렉토리 별로 용량을 제한하는 기능. ex4, XFS에서 사용 가능
  3. 서브 볼륨 쿼터: 파일시스탬 내의 서브 볼륨이라는 단위별 용량을 제한하는 기능. 디렉토리 쿼터와 유사. Btrfs에서 사용 가능.

파일 시스템이 깨지는 경우
------------------------

- 파일 시스템에서 이루어지는 작업은 *atomic* 해야한다. 데이터를 수정할 때 문제가 생겨서 atomic하게 작업이 이루어지지 않는 경우를 위해서 여러 기술이 적용.

#### 저널링

- ext4, XFS에서 사용하는 기술
- 파일 시스템 내에 저널이라는 영역을 갖는다. 저널 영역에 처리되는 내용(**저널 로그**)을 처리하기전에 작성. 처리된 내용을 작업
- 만약 파일 시스템에 오류가 생기는 경우에는 저널 로그를 처음부터 다시 수행

#### Copy on Write

- Btrfs에서 사용
- 오래된 파일시스템(ext4와 XFS)에서는 파일을 작성하면 그 파일의 배치 장소는 변함 없음. 파일의 위치가 변하는 경우에는 실제로 파일을 움직이지는 않고 메타데이터만 변경이 일어난다. 파일 내용이 변하는 경우에도 같은 위치에 업데이트.
- Btrfs에서는 파일을 작성하더라도 업데이트할 때마다 다른 장소에 파일 작성.
- 파일을 작성하다가 문제가 발생하면 기존의 파일에는 문제가 없기 때문에 작업을 다시 처음부터 수행하면 됨.


파일의 종류
--------

- 파일의 종류에는 일반 파일, 디렉토리, 디바이스 파일이 존재.
- 리눅스에서는 하드웨어의 네트워크 어댑터를 제외한 모든 장치를 파일로 표현해서 사용. 이 파일들에 open, read, write등의 시스템 콜을 이용해서 디바이스를 조작. 디바이스 파일들은 캐릭터 장치, 블록 장치로 나뉘며 */dev* 디렉토리 아래에 존재한다.
- 파일의 종류  
  - 일반(-)
  - 디렉토리(d)
  - 디바이스
    - 캐릭터(c): 읽기와 쓰기는 가능하지만 탐색(seek)이 불가능. ex) 터미널, 키보드, 마우스
    - 블록(b): 읽기,쓰기,랜덤 접근 가능. HDD, SSD 등의 저장장치가 존재. 블록장치는 보통 직접 접근하지 않고 파일 시스템을 작성한 후에 마운트 해서 사용.

```
[root@s181376e15b1 index0]# ls -l /dev/
합계 0
crw------- 1 root root     10, 235  6월  6 14:16 autofs
drwxr-xr-x 2 root root          80  6월  6 14:15 block
crw------- 1 root root     10, 234  6월  6 14:15 btrfs-control
drwxr-xr-x 3 root root          60  6월  6 14:15 bus
drwxr-xr-x 2 root root        2840  6월  6 14:16 char
crw------- 1 root root      5,   1  6월  6 14:16 console
lrwxrwxrwx 1 root root          11  6월  6 14:15 core -> /proc/kcore
drwxr-xr-x 4 root root          80  6월  6 14:15 cpu

//터미널에 write
[root@s181376e15b1 index0]# ps -aux | grep bash
root      1279  0.0  0.0 116716  3436 pts/0    Ss   09:53   0:00 -bash
root     10424  0.0  0.0 113288  1208 ?        Ss    6월06   0:00 bash /root/start_airflow.sh
root     21259  0.0  0.0 116984  1028 pts/0    S+   12:07   0:00 grep --color=auto bash
[root@s181376e15b1 index0]# echo hello > /dev/pts/0
hello
```

메모리 기반 파일시스템
-------------

- tmpfs: 저장 장치 대신 메모리에 작성하는 파일 시스템. 휘발성이지만 속도가 빠름. 디바이스 드라이버를 거치지 않는다.

```
[root@s181376e15b1 ~]# mount
~~ 생략 ~~
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
~~ 생략 ~~

//shared가 tmpfs에 의해 사용되는 메모리 양을 표시한다.
[root@s181376e15b1 ~]# free
              total        used        free      shared  buff/cache   available
Mem:        3870480     1072132     2441880      197184      356468     2404396
Swap:             0           0           0
```

네트워크 파일시스템
-------------

- nfs: 네트워크를 통해 연결된 다른 호스트에 있는 파일에 접근하는 파일 시스템.

가상 파일시스템
-----------

- 커널의 동작을 변경하거나 커널안의 정보를 얻기 위한 파일시스템

#### procfs

- 시스템에 존재하는 **프로세스**에 대한 정보를 얻기 위한 파일시스템
- */proc* 에 마운트 되며 */proc/{pid}/* 이하의 파일에 접근하여 정보를 얻을 수 있다.

```
[root@s181376e15b1 ~]# ps -aux | grep bash
root     25042  0.0  0.0 116608  3128 pts/0    Ss   20:55   0:00 -bash
root     26296  0.0  0.0 116984  1028 pts/0    S+   21:05   0:00 grep --color=auto bash
[root@s181376e15b1 ~]# cat /proc/25042/maps => 프로세스의 메모리 맵
00400000-004de000 r-xp 00000000 ca:01 67161943                           /usr/bin/bash
~~생략~~
[root@s181376e15b1 ~]# cat /proc/25042/cmdline => 프로세스 명령어 라인 파라미터
-bash
[root@s181376e15b1 ~]# cat /proc/25042/stat => 프로세스의 상태. 지금까지 사용한 CPU 시간, 우선도, 메모리의 양
25042 (bash) S  ~생략~

[root@s181376e15b1 ~]# cat /proc/cpuinfo  => 시스템의 CPU에 대한 정보
[root@s181376e15b1 ~]# cat /proc/diskstats => 시스템의 저장 장치에 대한 정보
[root@s181376e15b1 ~]# cat /proc/meminfo => 시스템의 메모리에 대한 정보
[root@s181376e15b1 ~]# cat /proc/sys/ => 커널의 각종 튜닝 파라미터. sysctl로 변경
````

#### sysfs

- procfs를 남용하는 것을 방지하기 위해 커널과 시스템에 대한 정보를 저장하는 파일시스템
- */sys/devices/*: 시스템에 탑재된 디바이스에 대한 정보들
- */sys/fs/*: 시스템에 탑재된 파일시스템에 대한 정보들

#### cgroupfs

- cgroup: 프로세스들로 이루어진 그룹에 대하여 여러 리소스(CPU, memory, fs, network device 등)의 사용량에 재한을 가하는 기능.
- cgroupfs는 cgroup을 다루는 파일 시스템. */sys/fs/cgroup/* 아래에 마운트 된다.
  - */sys/fs/cgroup/cpu/*: 그룹의 cpu의 사용량 제한.
  - */sys/fs/cgroup/memory/*: 그룹의 memory의 사용량 제한.

저장 장치
=======

I/O 스케쥴러
-----------

- 블록 장치 계층의 I/O 스케쥴러 기능은 블록 장치에 접근하는 요청을 일정 기간만큼 모아뒀다가 merge(여러개의 연속된 섹터에 대한 I/O 요청을 한 번에 합친다) -> sort(여러 개의 불연속적인 섹터에 대한 I/O 요청을 섹터 번호 순으로 정리)후에 한 번에 처리

- - -

네트워크
======

iptables
---------

- iptables: 리눅스에서 *firewall*(방화벽)을 설정하는 도구. 커널의 netfilter의 packet filtering 기능을 user mode에서 제어하는 수준으로 사용 가능. 패킷에 대해 허용(ACCEPT)과 차단(DROP)을 Chain으로 결정.
  - packet filtering: packet(사실 L4니까 프레임이 맞다)의 header를 보고 packet의 행선지를 결정하는 기능. header에는
  - Chain: 패킷이 조작될 상태를 지정. 기본 Chain은 삭제 불가능.
    - Chain INPUT: 서버로 들어오는 패킷에 대한 기본 정책
    - Chain FORWARD: 서버에서 포워딩(거쳐가능)되는 패킷에 대한 기본 정책
    - Chain OUTPUT: 서버에서 나가는 패킷에 대한 기본 정책
  - Masquerade: 사설 IP의 PC들이 외부 인터넷과 연결 가능할 수 있게 해주는 기능
  - NAT(Networ Address Translation)
    - SNAT(Source NAT): 내부 사설 IP에서 외부로 나갈 때 공인 IP로 변환. Masquerade와 비슷.
    - DNAT(Destination NAT): 외부에서 방화벽으로 오는 주소. 내부 사설 IP로 변환

- 현재 본인의 리눅스 서버의 방화벽을 iptables로 확인해보자. 기본 정책은 INPUT, OUTPUT은 허용, FORWARD는 차단이다.

```
[root@s181376e15b1 ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy DROP)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

- 모든 방화벽은 먼저 등록된 것부터 수행한다. 즉, 등록 순서가 매우 중요하다. 모든 입출력에 대해 차단을 설정하면,  그 후에 어떤 허용 설정이 등록되더라도 모든 패킷이 차단된다.

route
-----

- routing table을 설정하는 유틸. 라우팅 테이블은 현재 시스템의 네트워크 패킷이 다른 네트워크로 패킷을 전달할 때 어떤 네트워크 인터페이스에 보내야하는지를 설정한 테이블이다.

```
[root@s181376e15b1 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth0
10.41.8.0       0.0.0.0         255.255.254.0   U     0      0        0 eth0
link-local      0.0.0.0         255.255.0.0     U     1002   0        0 eth0
```

- 위의 명령어를 실행시키면 알 수 있는 정보는 다음과 같다
  1. Destination: 목적지 주소
  2. Gateway: 외부 네트워크와 연결하기 위한 게이트웨이 주소
  3. Genmask: Destination의 netmask. 0.0.0.0은 기본 게이트웨이 주소.
  4. Flag: 해당 경로에 대한 정보. U(up)은 살아있는 상태, H(host)는 목적지가 host 주소, G(gateway)는 게이트웨이를 의미
  5. Metric: 목적지까지의 거리
  6. Ref: 경로를 참조한 횟수
  7. Use: 경로를 탐색한 횟수
  8. Iface: 패킷을 보낼 네트워크 인터페이스
- 보통 *gateway*는 라우터를 의미한다. default gateway는 os에서 자동으로 찾기 때문에 연결되어 있는 공유기가 될 수 도 있다.

127.0.0.1 vs Localhost
---------------------

- 127.0.0.1: loopback interface에 할당된 IP 주소. 이 주소로 패킷을 전달하면 네트워크 계층은 외부로 전달하지 않고 자신이 받은 것 처럼(루프백) 행동.
- localhost: 보통 127.0.0.1의 주소를 갖는 host의 도메인 네임. locahost를 사용하면 resolve를 해야한다. localhost는 127.0.0.1이 아닌 다른 루프백 주소로도 resolve되기도 한다. resolve는 os의 redirect rule에 의해 보내진다.
- 127.0.0.1에 매핑된 정보는 */etc/hosts* 파일에 적혀있다.

```
[root@s181376e15b1 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

- 여러 방법으로 80포트에 매핑되어있는 nginx에 요청을 보내보자

```
[root@s181376e15b1 ~]# docker run -d -p 80:80 --name nginx  nginx:latest
a03e7c83967ad11a775e7544771a6ee831e5d407f610eabd2c3664dd78fc6deb
[root@s181376e15b1 ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                NAMES
a03e7c83967a   nginx:latest   "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:80->80/tcp   nginx

[root@s181376e15b1 ~]# curl 127.0.0.1:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

[root@s181376e15b1 ~]# curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

[root@s181376e15b1 ~]# curl localhost.localdomain:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

[root@s181376e15b1 ~]# curl localhost4:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

네트워크 인터페이스
---------

- 네트워크 하드웨어랑 통신하기 위한 소프트웨어 interface. 물리적, 논리적 2가지로 구별된다
  - Physical NI: NIC와 같은 실제 하드웨어 네트워크 디바이스를 의미. ex) eth0은 Ethernet inteface card를 의미
  - Virtual NI: 실제 하드웨어 네트워크 디바이스를 의미하지는 않는다. 보통 다른 NI에 연결하는 역할. ex) loopback, bridges, vlan, veth 등

- *ifconfig*는 netword interface 정보를 얻을 수 있는 명령어

```
[root@s181376e15b1 ~]# ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
~ 생략 ~

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
~ 생략 ~

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
~ 생략 ~
```

- 위에서의 정보에 대한 설명
  - eth0: Ethernet network card를 의미하는 물리적 인터페이스. 인터넷과 다른 호스트에 연결하기 위해 사용됨
  - lo: loopback이라고 불리는 가상 네트워크 인터페이스.
  - docker0: Docker에서 사용되는 가상 네트워크 인터페이스. Bridge로 되어있다.

[참고자료](https://codewithyury.com/demystifying-ifconfig-and-network-interfaces-in-linux/)


가상 네트워크 인터페이스
-----------------

- Brdige: 일반적인 네트워크 switch와 비슷하게 동작. 라우터, 게이트웨이, VM, 컨테이너 등에서 패킷을 목적지로 forwarding.
- Bonded interface: 서로 다른 네트워크 인터페이스를 하나의 논리적 인터페이스로 bond하는 기능.
  - stand-by mode: 네트워크 인터페이스를 Active와 Standby로 나누고, Active에서 장애발생시 트래픽을 Standby로 전달.
  - load balancing mode: 패킷을 여러 네트워크에 어떤 규칙에 의해서 분배하는 기능. RR, XOR, Broadcat등의 정책 가능
- VLAN(virtual LAN): 하나의 스위치에서 broadcast 도메인을 논리적으로 분할. 브로드캐스트는 동일한 LAN에 모든 호스트에 패킷을 전송. VLAN은 LAN을 논리적으로 분할하며 브로드캐스팅에 의해 대역폭의 불필요한 낭비를 감소시킨다.
- VETH(virtual ethernet): 한 쌍으로 생성되는 로컬 이더넷. 한 쪽에서 다른 쪽으로 패킷 전송.
