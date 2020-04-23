---
layout: post
title:  "Network"
date:   2019-12-18 01:02:59
author: Inhyuk
name: linux_network.md
category: linux
---

ifconfig
--------

- 리눅스에서는 네트워크 장치 이름을 ens32 or ens33으로 설정

```
//정보 출력
ifconfig [ens32|ens33]
//네트워크 장치 정지
ifdown [ens32|ens33]
//네트워크 장치 시작
ifup [ens32|ens33]
```

nslookup
-------

- DNS 서버의 작동을 테스트

ping
-----

- 해당 컴퓨터가 응답하는지를 테스트하는 명령어
- ping <ip주소|URL>

nm-connection-editor
-----------------------

- 네트워크와 관련된 대부분의 작업을 수행
- ip주소, 서브넷마스크, 게이트웨이 정보 입력
- DNS 정보 입력
- 네트워크 카드 드라이버 설정
- 네트워크 장치 설정
- GUI 기반
- CLI는 nmcli 사용, text기반은 nmtui 사용

- systemctl <start|stop|restart|status> network : 네트워크 설정 변경 후 시작, 정지, 재시작, 상태 확인

nmcli
-----

- [참고자료1](https://mansoo-sw.blogspot.com/2016/10/linux-networkmanager-console-nmcli.html)
- [참고자료2](https://haker.tistory.com/55)
- device : 물리적인 장치. 한개 이상의 연결을 가질 수 있음.
- connection : device에 연결하여 유무선 네트워크 관리. ip4, 암호, 연결 등등을 관리
- 새로 생성된 connection의 정보는 /etc/NetworkManager/system-connections/연결이름 파일에 적혀있다.


```
//설치
sudo apt install network-manager

//시작
systemctl start network-manager
```

- 설치 후 시작을 하면 /etc/network/interfaces를 바로 관리하는 것이 아니다.
- /etc/NetworkManager/NetworkManager.conf의 [ifupdown]를 managed=true로 설정하자


```
//재시작
systemctl start network-manager

//확인
nmcli con show

//장치 상태 확인
nmcli dev show

//활성화된 연결 확인
nmcli con show --active

//연결 생성
nmcli con add [타입, 이름, ifname]

//설정 활성화
nmcli con up 이름

//설정 비활성화
nmcli con down 이름

//설정 리로드
nmcli con reload

//연결 삭제
nmcli con delete

//연결 수정
nmcli con mod 이름 수정내용

```

### 예제

현재의 상태를 확인해보자

![상태확인]({{site.baseurl}}/post_img/{{page.name}}/nmcli_con.png)

ens33에 연결되어 있다.

새로운 정적 연결을 생성하자

![추가]({{site.baseurl}}/post_img/{{page.name}}/nmcli_static.png)

연결이 만들어졌지만 활성화는 되지 않은 상태다

![활성화]({{site.baseurl}}/post_img/{{page.name}}/nmcli_up.png)


설정 파일
---------

- nmtui, nm-connection-editor, nmcli : /etc/NetworkManager/system-connections 밑에 연결 설정 저장
- /etc/network/interfaces에 설정파일이 존재. network-manager로 관리하자.

- DNS 설정파일은 /etc/resolv.conf에 임시로 저장.
- /etc/hosts에 호스트이름, FQDN 등이 들어 있음
