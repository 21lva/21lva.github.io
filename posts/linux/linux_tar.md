---
layout: post
title:  "각종 명령어 정리"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_tar.md
category: linux
---

출력
=======

- cat 파일 : 파일의 내용을 출력.
- cat 파일1 파일2 : 연속해서 출력
  - -n : 행 번호를 붙여서 출력(빈행도)
  - -b : 행 번호를 붙여서 출력
  - -s : 연속되는 2개의 빈 행은 한 행으로 출력

- head -숫자 파일 : 앞의 숫자만큼 줄 출력 (기본 10줄)
- tail -숫자 파일 : 뒤의 숫자만큼 출력(기본 10줄). 로그 파일에 주로 사용.

- more : 페이지 단위로 보여줌.
  - space bar (f): 한 화면씩 아래로
  - enter : 한 행씩 아래로
  - b : 한 페이지씩 다시 위로
  - = : 현재 위치의 행 번호 표시
  - /문자열 : 지정한 문자열 검색
  - q : 종료
  - !쉘명령어 : more상태에서 쉘 명령어 실행
  - v : 현재 위치에서 vi 실행

- less : 페이지 단위로 보여줌. more + 몇 가지 기능. 표식 가능
  - G : 파일의 맨 마지막으로 이동
  - g : 맨 처음으로 이동
  - page up , b : 한 페이지 위로
  - page down, space bar : 한페이지 아래로

- - -

grep
======

- [참고자료1](https://recipes4dev.tistory.com/157)
- [참고자료2](http://www.dreamy.pe.kr/zbxe/CodeClip/6331)
- 출력 문자열 중에 원하는 단어나 글자가 들어 있는 라인만 출력
- find, ps, cat 등등 + 파이프(|, 결과를 다른 프로그램의 입력으로 연결) 와 같이 사용 하기도 함.
- grep [option] [pattern] [file]

- option
  - -E : PATTERN을 확장 정규 표현식(Extended RegEx)으로 해석.
  - -F : PATTERN을 정규 표현식(RegEx)이 아닌 일반 문자열로 해석.
  - -G : PATTERN을 기본 정규 표현식(Basic RegEx)으로 해석.
  - -P : PATTERN을 Perl 정규 표현식(Perl RegEx)으로 해석.
  - -e : 매칭을 위한 PATTERN 전달.
  - -f : 파일에 기록된 내용을 PATTERN으로 사용.
  - -i : 대/소문자 무시.
  - -v : 매칭되는 PATTERN이 존재하지 않는 라인 선택.
  - -w : 단어(word) 단위로 매칭.
  - -x : 라인(line) 단위로 매칭.
  - -z : 라인을 newline(\n)이 아닌 NULL(\0)로 구분.
  - -m : 최대 검색 결과 갯수 제한.
  - -b : 패턴이 매치된 각 라인(-o 사용 시 문자열)의 바이트 옵셋 출력.
  - -n : 검색 결과 출력 라인 앞에 라인 번호 출력.
  - -H : 검색 결과 출력 라인 앞에 파일 이름 표시.
  - -h : 검색 결과 출력 시, 파일 이름 무시.
  - -o : 매치되는 문자열만 표시.
  - -q : 검색 결과 출력하지 않음.
  - -a : 바이너리 파일을 텍스트 파일처럼 처리.
  - -I : 바이너리 파일은 검사하지 않음.
  - -d : 디렉토리 처리 방식 지정. (read, recurse, skip)
  - -D : 장치 파일 처리 방식 지정. (read, skip)
  - -r : 하위 디렉토리 탐색.
  - -R : 심볼릭 링크를 따라가며 모든 하위 디렉토리 탐색.
  - -L : PATTERN이 존재하지 않는 파일 이름만 표시.
  - -l : 패턴이 존재하는 파일 이름만 표시.
  - -c : 파일 당 패턴이 일치하는 라인의 갯수 출력.

- 메타 문자
  - \. 	: 1개의 문자 매치 (정확히 1개의 문자와 매치)
  - \* 	: 앞 문자가 0회 이상 매치
  - {n} :	앞 문자가 정확히 n회 매치
  - {n,m} :	앞 문자가 n회 이상 m회 이하 매치
  - [ ] :	대괄호에 포함된 문자 중 한개와 매치
  - [^ ] :	대괄호 안에서 ^뒤에 있는 문자들을 제외
  - [ - ] :	대괄호 안 문자 범위에 있는 문자들 매치
  - () : 표현식을 그룹화
  - ^ :	문자열 라인의 처음
  - $ :	문자열 라인의 마지막
  - ? :	앞 문자가 0 또는 1회 매치 (확장 정규 표현식)
  - + :	앞 문자가 1회 이상 매치 (확장 정규 표현식)
  - | :	표현식 논리 OR (확장 정규 표현식)

- example

```
//하위 디렉토리를 포함한 모든 파일에서 문자열 검색
grep -r "pat" *

//검색 결과 앞에 파일 이름 표시
grep -H "pat" *

//step0 ~ step9 찾기
grep step[0-9] file.txt

//. 로 끝나는 문자열 검색
grep "\.$" file.txt
```

- - -

redirection
===========

- 표준입출력의 방향을 바꿔줌
- 명령 > 파일 : 명령의 결과를 파일로 저장
- 명령 >> 파일 : 명령의 결과를 파일에 추가
- 명령 < 파일 : 파일의 데이터를 명령에 입력

```
cat < filename
cat file1 >> file2
```
- - -

alias
-----

- 자주 사용하는 명령어를 간단하게 설정
- unalias로 해제
- alias 새로운명령어='command' : command를 새로운 명령어로 만듬
- alias : alias 목록 출력

- - -

ls
====

- 목록 보기(기본 : 알파벳 순서로 출력)

- option
  - -l : 자세한 내용 출력. 수정시간을 출력.(첫글자 d : 디렉토리, b: 블록특수파일, c : 문자특수파일, l : 심볼릭 링크, p : 선입선출 특수 파일, s : 로컬 소켓, - : 일반 파일)
  - -a : 숨긴 파일, 디렉토리도 보여줌
  - -S : 파일 크기 순으로 정렬하여 보여줌
  - -r : 거꾸로 출력(알파벳 역순)
  - -R : 하위 디렉토리까지 출력
  - -h : 사람이 보기 좋게 K,M,G 단위 사용하여 보여줌
  - -lu : -l 수행시 수정시간이 아닌 접근시간으로 보여줌
  - -lc : 변경시간으로 보여줌
  - -d : 디렉토리 정보만 보여줌
  - -F : 디렉토리의 경우 /, 실행가능한 경우 별표 , 소켓인 경우 =, 심볼릭 링크인 경우 @ 를 붙여서 보여줌
  - -m : 쉼표로 구분해서 보여줌
  - -i : inode 번호 출력

- - -

cd
===

- 이동
- cd ~ : 자신의 홈 디렉토리로 이동
- cd ~사용자이름 : 사용자의 홈 디렉토리로 이동
- cd $변수 : 변수명에 저장된 디렉토리로 이동
- cd - : 이전 경로로 이동

- - -

touch
======

- 빈 파일을 생성하거나 time stamp를 바꾸는 용도
- touch 파일이름 : 파일이 없을 경우 빈 파일 생성. 있을 경우 현재 시간으로 atime(최종 접근 시각),mtime(최종 수정 시각),ctime(최종 상태 변경시각) 변경
- touch -t [시간] 파일이름 : atime과 mtime을 제공한 시간으로 갱신
- -c : 파일이 없으면 새로운 파일을 생성하지 않는다

- - -

cp
=======

- 복사
- cp 원본 복사본 : 원본 파일로 복사본을 생성
- cp 원본1 원본2 원본3 디렉토리/ : 원본 파일들을 디렉토리 안에 복사
- cp -r 원본디렉토리 복사디렉토리

- option
  - -b : 백업파일 생성
  - -f : 지우고 복사
  - -i : 덮어씌울 것인지 물어본다. 기본으로 되어 있음
  - -u : 이미 파일이 존재하면 수정 날짜를 비교하여 최신일 경우에만 복사
  - -r , -R : 디렉토리 복사
  - -S : 동일한 이름의 파일이 존재하면 백업파일을 생성하고 그 끝에 붙여질 접미사를 지정한다.
  - -p : 복사되는 파일이 원본파일과 모드, 소유자, 시간정보가 같게 한다.
  - -v : 복사 진행 작업을 표시

- - -

rm
====

- 삭제

- option
  - -r : recursive. 디렉토리를 포함한 내용들을 지울때 사용
  - -i : 물어보고 지움
  - -f : 물어보지 않고 강제 삭제

- - -

mv
======

- 이동 또는 이름 재지정
- mv 원본파일 이동위치or이름

- option
  - -b , --backup : 백업후 이동
  - -f : 덮어 씌우기
  - -i : 덮어 씌울 때 물어본다
  - -S , --suffix=SUFFIX : 백업파일 생성시 지정된 접미사를 붙여서 생성.
  - -n : 동일한 이름이 있으면 이동하지 않는다
  - -u : 이미 파일이 존재하면 최신일 경우에만 덮어씌우기로 이동.
  - -v : 진행사항 출력

- rename 해당사항 변경사항 대상파일 : 대상파일에서 해당사항을 변경사항으로 바꿔준다. 여러 파일이 한번에 바꿔진다.
  - rename ab xy ab* : ab을 접두사로 가지는 파일을 xy를 접두사로 가지는 파일로 이름 변경.
  - rename ab xyz ab? : ab를 가지고 뒤에 글자가 하나 오는 파일들의 이름을 xy어쩌고로 변경.

- - -

find
=====

- find [경로] [옵션] [조건] : 파일 찾기
- 여러 조건 연속적으로 사용 가능

- 아래는 EXT를 확장자로 가지는 파일들을 현재 디렉토리 기준으로 찾고 삭제하는 명령어다

```
find . -name "*.EXT" -delete
```

- option
  - -P : 심볼릭 링크를 따라가지 않고, 자체 정보 사용
  - -L : 심볼릭 링크에 연결된 파일 정보 사용
  - -D : 디버그 메시지 출력

- 조건
  - -name 문자열: 문자열 패턴에 해당하는 파일 검색
  - -empty : 빈 디렉토리 또는 크기가 0 인 파일 검색
  - -delete : 검색된 파일 또는 디렉토리 삭제
  - -exec 명령문: 검색된 파일에 대해 지정된 명령 실행. 검색된 파일은 {}로 명령문에 쓸 수 있음. 명령문 끝에는 \\; 사용
  - -path 문자열: 지정된 문자열에 해당하는 경로 검색
  - -print : 검색 결과 출력(newline으로 구분). 기본값
  - -print0 : 검색 결과 출력(줄바꿈 없음)
  - -size 숫자 : 파일 크기를 사용하여 파일 검색. G,M,K 등 사용. +,- 를 통해서 이상 이하 가능.
  - -type 타입 : 해당하는 파일 타입 검색(-type d 등등)
  - -mindepth 숫자: 하위 디렉토리 최소 깊이 지정
  - -maxdepth 숫자: 최대 깊이 지정
  - -atime : 접근 시각을 기준으로 검색
  - -ctime : 내용 및 속성 변경 시각을 기준으로 검색
  - -mtime : 데이터 수정 시각을 기준으로 검색
  - -newer 파일 : 주어진 파일보다 최근 파일 검색

```
1. find . -empty -exec ls -l {} \;
2. find . -empty -exec rm {} \;
3. find . -name [FILE] 2> /dev/null
```
3의 경우 에러메시지를 출력하지 않는다

- - -

tar
====

- [잘 나와있는 참고자료](https://recipes4dev.tistory.com/146)
- 여러 파일을 한번에 묶거나, 묶거나 압축할 때 사용
- 반대 과정도 가능
- 원칙적으로는 묶기만함(압축x)
- But 옵션을 통해서 이런 기능 제공(gzip, bzip2 등을 사용하지 않아도 됨. 하지만 이 기능들이 설치되어 있어야 함.)
- tar [옵션] [나올 파일] [대상파일|파일들 경로|파일 나열]

- option : hypen(-)는 쓰지말자 오류가 날 수 있다.[관련 내용](https://unix.stackexchange.com/questions/149494/tar-exits-on-cannot-stat-no-such-file-of-directory-why)
  - f : 대상 tar 아카이브 지정. (기본 옵션)
  - c : tar 아카이브 생성. 기존 아카이브 덮어 쓰기. (파일 묶을 때 사용)
  - x : tar 아카이브에서 파일 추출. (파일 풀 때 사용)
  - v : 처리되는 과정(파일 정보)을 자세하게 나열.
  - z : gzip 압축 적용 옵션.
  - j : bzip2 압축 적용 옵션.
  - t : tar 아카이브에 포함된 내용 확인.
  - C : 대상 디렉토리 경로 지정.
  - A : 지정된 파일을 tar 아카이브에 추가.
  - d : tar 아카이브와 파일 시스템 간 차이점 검색.
  - r : tar 아카이브의 마지막에 파일들 추가.
  - u : tar 아카이브의 마지막에 파일들 추가.
  - k : tar 아카이브 추출 시, 기존 파일 유지.
  - U : tar 아카이브 추출 전, 기존 파일 삭제.
  - w : 모든 진행 과정에 대해 확인 요청. (interactive)
  - e : 첫 번째 에러 발생 시 중지.

![tar]({{site.baseurl}}/post_img/{{page.name}}/tar.png)

- - -

zip
===

- 다른 os와의 호환을 위한 것
- zip 압축파일.zip [압축할 폴더|파일] : 압축
  - -r : recursive
  - -F : 한글이름
- unzip 파일이름.zip : 현재에 압축해제
- unzip 파일.zip -d 경로 : 경로에 압축해제

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

- - -


dpkg
====

- debian package
- redhat의  rpm과 유사

- option
  - --info(-I) <deb 파일>: 패키지 정보 확인
  - --contents(-c) <deb 파일>: 구성 파일 리스트
  - -i <deb 파일> : 설치
  - --unpack <deb 파일> : 압축 해제(설치는 하지 않음)
  - --remove <패키지> : 패키지를 설정파일은 유지하고 제거
  - --purge(-P) <패키지> : 패키지를 설정파일까지 제거
  - -l : 설치된 패키지 리스트 확인
  - -L <패키지> : 설치된 패키지에 포함된 파일 보기
  - -S <deb 파일> : 파일의 패키지명 알아내기
  - -s <패키지> : 패키지에 대한 상태 확인
  - -p <패키지> : 패키지에 대한 정보 확인


- - -

apt
====

- advanced packaging tool
- dpkg의 설치 기능 + 검색, 다운로드, 업그레이드, 검사 등을 진행

- apt update : /etc/apt/sources.list 를 참조하여 사용할 수 있는 패키지 DB를 업데이트. /var/lib/apt/lists에 업데이트
- apt search 키워드 : 키워드를 대소문자 구분 없이 검색하여 키워드를 포함하고 있는 패키지를 검색
- apt purge 패키지: 패키지와 관련 설정 삭제
- apt remove 패키지: 설정파일을 제외한 패키지 삭제
- apt install 패키지: 패키지 설치
  - -y : 설치 여부 질문에 긍정응답
  - --reinstall : 재설치
- apt upgrade : 업그레이드 가능한 모든 패키지 업그레이드
- apt full-upgrade : 의존성 고려한 패키지 업그레이드
- apt show 패키지 : 패키지 상제 정보 출력
- apt autormove : 사용하지 않는 패키지 제거
- apt list : 레포에 등록된 패키지 목록 조회
  - --installed : 설치된 패키지 목록 조회
  - --upgradable : 설치된 패키지 중 업그레이드 대상 패키지 조회
  - --all-versions : 패키지 모든 버전 목록 조회

- apt content 패키지 : 설치된 위치 확인
- apt depends 패키지 : 의존성 확인
- 저장소 추가
  - apt-add-repository ppa:<저장소 주소>
  - apt-add-repository <저장소 주소>

- 기본으로 제공되는 저장소를 변경하려면 /etc/apt/sources.list를 vim으로 열고 *%s/기존저장소/변경저장소* 를 한다.



- [apt와 apt-get의 차이](https://askubuntu.com/questions/481241/what-is-the-difference-between-sudo-apt-get-install-and-sudo-apt-install)
- 요약하면 **apt** 는 **apt-get** 와 **apt-cache** 의 명령어를 포함한다고 한다(wrapper)

- [apt vs aptitude](https://www.tecmint.com/difference-between-apt-and-aptitude/)
- aptitude가 더 high-level 이라고 한다. 삭제를 할때 --auto-remove 기능을 포함한다고 하나?

- - -


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
