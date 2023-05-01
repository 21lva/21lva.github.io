---
layout: post
title:  "dpkg, apt"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_dpkg.md
category: linux
---

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
