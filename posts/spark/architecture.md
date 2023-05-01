---
layout: post
title:  "Spark 구조"
date:   2022-01-22 02:02:59
author: Inhyuk
category: spark
tags: spark
cover:  "/assets/instacode.png"
name: architecture.md
---

스파크 구조
========

- 스파크는 크게 client, driver, executor로 구성됨. 각각의 실제 물리적 위치는 어떤 cluster위에서 동작하냐에 따라 달라짐. client와 executor는 하나의 jvm위에 하나씩 존재
  - client: driver 프로세스를 시작하는 역할. ex) spark-submit 스크립트
  - driver: spark application을 모니터링, 관리하는 jvm 프로세스. 스케쥴러, 컨텍스트를 감싸는 wrapper. 클러스터 매니저에 executor프로세스 실행을 위한 메모리와 cpu를 할당 요청. job을 stage와 task로 분할(stage는 task의 병렬 집합). executor에 task 전달.
    - context: driver에 의해 생성. RDD 생성, 데이터 로드등에 필요한 메서드 제공.
  - executor: driver가 보낸 task들을 실행하는 jvm 프로세스. 여러 task들을 task slot에서 꺼내서 병렬 처리. 하나의 executor는 다른 application과 공유되지 않는다. driver로 부터 전달받은 task들을 역직렬화하고 수행

- 각 job은 shuffle이라는 동작을 기준으로 여러 stage로 나뉜다. stage는 각각의 executor에서 병렬로(데이터 단위) 수행되는 job으로 나뉜다.
- shuffle: 파티션의 데이터가 다른 파티션으로 전달되는 과정. partition은 데이터를 분할한 것들이라고 생각하면 됨
  - join, distinct, groupBy 등의 단계에서 수행됨
  -  shuffle write(데이터 전송 이전에 일어나는 Map과정. 각 partition을 key-value형태로 os cache에 저장) -> transfer -> shuffle read(데이터 전송 이후에 일어나는 reduce 과정)
- task 종류:
  - shuffle map task: task중 마지막에 shuffle로 끝나는 task. job의 중간 단계들
  - result map task: task중 마지막에 결과를 driver에 전달하는 task. job의 마지막에 존재

리소스 스케쥴링
------

- executor에 리소스를 할당하는 과정
- cluser모드: cluster manager는 클라이언트로부터 전달 받은 driver process 시작 -> cluster manager는 driver가 요청한 executor프로세스를 실행(리소스 할당) -> executor가 종료되면 리소스 회수

잡 스케쥴링
--------

- executor와 driver(스케쥴러)가 통신하면서 task를 어떻게 분배할 것인지를 결정하는 과정. cluster manager와 관련 없이 동작한다.
- CPU를 task단위로 관리하면서 task 할당.
- spark context는 여러 스레드와 여러 사용자가 동시에 사용 가능하다(thread-safe). 이 때 여러 잡들은 동일한 context에서 만든 executor를 가지고 경쟁하는 구조. 동일한 context를 공유하기때문에 공유변수, 누적변수에 동시에 접근 가능
- DAG scheduler와 Task scheduler에 의해 동작
  - DAG scheduler: DAG를 stage와 task로 분할.
  - Task scheduler: task를 executor에 할당 -> executor에게 task실행 메세지 전송 -> executor가 해당 task 수행


데이터 지역성
---------

- 스파크는 **최대한** 데이터와 가까운 executor에서 task를 수행하도록 job scheudling
- 스파크는 각 partition별로 preferred location 목록을 가지고 있음.
- preferred location: patition의 데이터를 저장한 hostname 또는 executor 목록. HDFS로 생성된 데이터에서만 사용.
- 지역성 단계: 각 지역성 단계를 기다리다 일정 시간이 지나면 다음 단계로 넘어가서 수행 가능한지 확인
  - PROCESS_LOCAL: 파티션을 cache한 executor가 task 수행
  - NODE_LOCAL: partiton에 직접 접근 가능한 node에서 task 수행. 즉, 같은 물리적 머신에 있는 경우
  - RACK_LOCAL: YARN에서 가능. cluster의 RACK 정보를 참고할 수 있는 경우에 같은 RACK에 있는 executor에서 수행. RACK은 다양한 서버 및 네트워크 장비를 장착하는 표준 프레임.
  - NO_PREF: 선호위치 없음. 클러스터 어디에서나
  - ANY: 데이터 지역성을 보장하지 못하면 아무 위치에서나 수행

메모리 스케쥴링
-----------

- executor jvm의 메모리는 클러스터 매니저가 할당 -> 이 jvm메모리내에서 스파크가 각 job과 task에 메모리 할당.
- 1.5.2 이하의 스파크에서는 스파크가 할당받은 메모리 일부를 캐시와 셔플링 메모리로 할당.
- 1.6.0부터는 실행 메모리 영역과 스토리지 메모리 영역을 통합해 관리.



- - -

자체 클러스터
======

- 자체 클러스트의 시작 과정(node가 n개 인 경우)
  1. client가 master jvm에 application 제출
  2. master jvm이 1번 worker jvm에 driver 시작 명령
  3. 1번 worker jvm이 driver jvm 생성
  4. master가 모든 worker jvm에 executor jvm 시작 명령 -> worker들이 jvm 생성
  5. driver process는 executor와 직접 통신하면서 spark application 수행

- - -

YARN
====

- 하둡의 MR 실행 엔진. 하둡의 cluster manager
- spark는 cluster manager에 종속적이지 않지만, yarn에서만 있는 특징이 존재
- yarn에서 application들은 컨테이너에서 실행됨.
- 리소스 매니저: 클러스터당 하나 실행되는 매니저
- 노드 매니저: 클러스터 내의 노드 하나당 실행되는 요소.
- 어플리케이션 마스터: 클러스터에서 executor에 필요한 리소스와 executor 컨테이너 시작을 요청하는 역할. 클러스터 배포 모드에서는 컨테이너에서 실행되는 driver process의 wrapper로 리소스 요청과 함께 task 요청등을 수행.
- 클러스터 배포 모드에서의 스파크 시작 과정
  1. 클라이언트가 application을 리소스 매니저에 제출
  2. 리소스 매니저는 노드 매니저중 하나를 선택해서 어플리케이션 마스터가 실행될 컨테이너 실행을 명령
  3. 컨테이너에서 실행된 어플리케이션 마스터는 executor가 실행될 컨테이너들을 리소스 매니저에 요청
  4. 리소스 매니저가 리소스 할당을 승인 -> 어플리케이션 마스터가 executor container를 시작하라고 각각의 노드매니저에 요청 -> 노드매니저가 실행(이후로는 executor와 am이 직접 통신)
- 클라이언트 배포 모드에서의 스파크 시작 과정
  1. 클라이언트가 application을 리소스매니저에 제출
  2. 리소스 매니저는 노드 매니저중 하나를 선택해서 어플리케이션 마스터가 실행될 컨테이너 실행을 명령
  3. 컨테이너에서 실행된 어플리케이션 마스터는 executor가 실행될 컨테이너들을 리소스 매니저에 요청
  4. 리소스 매니저가 리소스 할당을 승인 -> 어플리케이션 마스터가 executor container를 시작하라고 각각의 노드매니저에 요청 -> 노드매니저가 실행
  5. 클러스터 배포 모드와 다르게 driver가 어플리케이션 매니저에 존재하지 않기 때문에, executor와의 통신은 client에 존재하는 driver가 수행

- YARN 클러스터를 사용하는 경우에는 코어 개수로 기본 값이 아닌 다른 값을 설정해야 함. 기본적으로 실행자를 2개, 각 실행자에 CPU코어 하나를 할당하기 때문에 부족한 경우가 발생.
- YARN에서는 jvm heap과 별개로 컨테이너 자체의 메모리 영역을 추가 할당할 수 있음. jvm 프로세르를 실행하는 데에 필요한 메모리 영역. executor가 spark.executor.memory(executor의 heap 메모리) + spark.yarn.executor.memoryOverhead(추가 컨테이너 메모리)보다 큰 값을 사용하고자 하면 컨테이너가 종료됨. 문제를 미연에 방지하기 위해서는 spark.yarn.executor.memoryOverhead를 적당히 큰 값(1024MB 이상)로 설정하자.

- - -

Lifecycle
=======

SparkSession
----------

- spark application은 먼저 spark session 생성
- REPL에서는 자동으로 생성

SparkContext
-----------

- SparkSession 내부에 존재
- cluster와 연결을 담당하고 RDD 같은 저수준 API를 사용

논리적 명령
-------

- spark code는 transformation과 action으로 이루어짐
- Dataframe은 선언적 명령 사용
