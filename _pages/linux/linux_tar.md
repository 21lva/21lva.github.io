---
layout: post
title:  "tar, zip"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_tar.md
category: linux
---

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


zip
===

- 다른 os와의 호환을 위한 것
- zip 압축파일.zip [압축할 폴더|파일] : 압축
  - -r : recursive
  - -F : 한글이름
- unzip 파일이름.zip : 현재에 압축해제
- unzip 파일.zip -d 경로 : 경로에 압축해제
