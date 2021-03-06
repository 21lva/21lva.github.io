---
layout: post
title:  "기본 명령어, 파일 명령어"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_basicCmd.md
category: linux
---


기본
=====

- tab : 자동완성
- tab+tab : 자동완성 후보 목록 보여줌
- !! : 이전 명령 실행
- history : terminal에서 실행한 명령어 목록
- !숫자 : history에서 나온 명령어 해당 숫자에 맞게 실행
- clear : 터미널 청소
- file 파일: 파일 종류를 확인
- type : 명령어가 내부 명령어, 외부명령어, alias인지를 확인시켜줌
- su - 사용자 : 사용자로 변경

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
