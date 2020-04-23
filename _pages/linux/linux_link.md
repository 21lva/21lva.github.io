---
layout: post
title:  "inode, 링크"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_link.md
category: linux
---


inode
=====

- 유닉스 계통 파일 시스템에서 사용하는 자료 구조
- 파일과 디렉토리는 고유한 inode 번호 보유
- 사용자가 파일에 접근하면 파일 이름을 통해서 디렉토리 테이블에 저당된 inode 번호로 매핑됨 -> inode 번호를 통해서 inode에 접근
- inode들은 inode block이라는 공간에 모여있다.

- inode의 정보
  - 허가권
  - 링크 수
  - 소유자
  - 소유 그룹
  - 파일크기
  - 파일 주소
  - 최근 접근 정보
  - 최근 수정 정보
  - inode 수정 정보

- inode의 파일 주소를 통해서 실제 데이터에 접근
- **ls -il** 를 통해서 inode번호 확인 가능(맨 앞 숫자)


![sticky]({{site.baseurl}}/post_img/{{page.name}}/inode.jpg)

- - -

하드 링크
========

- 원본 파일의 inode에 직접 연결
- ln 원본파일 링크파일
- 파일만 생성 가능
- 원본파일이 삭제되어도 데이터 접근 가능
- 동일한 inode번호 보유

- - -


심볼릭 링크
==========

- 원본 파일에 연결
- 심볼릭 링크에 해당하는 inode가 생성되고 이 inode가 원본 파일을 가리킴
- 원본과 다른 inode 번호 보유
- 원본 삭제시 데이터 접근 불가
- ln -s 원본파일 링크파일

- - -

참고자료
=======

<https://www.leafcats.com/141>

<https://koromoon.blogspot.com/2018/05/inode-symbolic-link-hard-link.html>

<https://mrrootable.tistory.com/37>
