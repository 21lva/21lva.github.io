---
layout: post
title:  "구조"
date:   2021-10-23 01:02:59
author: Inhyuk
category: Airflow
tags:	airflow
cover:  "/assets/instacode.png"
name: archi.md
---


- airflow는 **webserver, scheduler, Metadata DB**로 구성
- airflow 2 에서는 DAG를 DB에 직렬화 하여 저장하기 때문에 webserver가 DAG 파일에 직접 접근할 필요 없이 DB의 직렬화된 정보를 읽을 수 있다

- - -

webserver
=========

- web UI를 제공하는 서버

- - -

Scheduler
========

- 스케쥴러의 역할
  1. DAG Processor: DAG파일 분석 -> 정보 추출 -> metadata DB에 저장
  2. Task Scheduler: 실행 가능한 task를 *waiting* 상태로 전환
  3. Task Executor: waiting task를 등록(*queued*)

- schedule된 workflow를 시작하고, executor에게 task를 수행하게 한다.
- DAG와 Task들을 모니터링 하다가 필요한 조건을 만족하면 처리될 수 있도록 DAG directory에 설정한다.

- 스케쥴러는 DAG에 설정되어 있는 **start_date**을 기반으로 **DAG RUN**을 생성하고 그 후로는 **schedule_interval**을 기준으로 주기적으로 생성
- **allow_trigger_in_future**: 외부 트리거를 사용하고자 하면 airflow.cfg의 설정을 true로 수정한다

- 멀티 스케쥴러 하에서 스케쥴러들은 직접 통신을 하지 않고, DB를 통해서 정보를 송수신한다.
- 다음은 스케쥴링 루프이다.
  - DagRun으로 생성이 필요한 DAG를 찾고 생성한다
  - DagRuns에서 스케쥴링 가능한 task instances와 DagRuns를 검사한다
  - 스케쥴링 가능한 task instances를 pool limit과 concurrency limit에 따라 실행열에 추가한다
- 스케쥴러는 성능을 위해서 in memory로 계산되는 스케쥴링 loop가 있으므로 **critiical section**에는 하나의 스케쥴러만 접근가능하도록 해야한다.
  - critical section은 task instances들이 실행되고 예약되는 부분으로 row-level lock으로 보호된다

DAG Processor
---------------

- airflow는 DAG directory(AIRFLOW__CORE__DAGS__FOLDER)의 파이썬 파일을 주기적으로 처리 -> 변경이 존재하면 metastore에 등록
  - AIRFLOW__SCHEDULER__PROCESSOR_POLL_INTERVAL: 스케쥴러가 루프를 완료한 후 대기하는 시간. 이 숫자가 낮은 DAG가 더 빨리 구문 분석됨
  - AIRFLOW__SCHEDULER__MIN_FILE_PROCESS_INTERVAL: 파일이 처리되는 최소 간격.
  - AIRFLOW__SCHEDULER__DAG_DIR_LIST_INTERVAL: DAG 폴더의 파일 리스트를 새로 고치는 최소 시간. 이미 등록된 파일은 다른 간격으로 처리됨
  - AIRFLOW__SCHEDULER__PARSING_PROCESSES: 모든 DAG파일을 구문 분석하는 데 사용할 최대 프로세스 수.

Task Scheduler
--------------

- 스케쥴러가 실행할 task instance를 결정하는 역할
- 각 테스크 인스턴스에 대해 모든 조건이 충족되는지를 확인하고 만족하는 경우에 테스크가 성공적으로 수행되었음을 인식한다. 그 이후에 다음 작업일정까지 **예약 상태**로 변경
  - 조건: 테스크의 종속성이 충족되었는지, 정상적으로 마지막까지 완료되었는지, 직전 DAG의 테스크 인스턴스가 depends_on_past=True 조건을 갖는지 등
- 해당 테스크를 위한 슬롯에 여유가 있거나 우선순위가 높으면 해당 테스크를 **예약 상태**에서 **대기 상태**로 변경. 이 후로는 테스크 스케쥴러가 아닌 익스큐터가 테스크를 다룬다

Executor
------

- 대기열에서 테스크 인스턴스를 가져와서 실행하는 역할.
- 실행되는 테스크 인스턴스의 상태에 대한 정보를 metastore에 저장하여 반영한다. airflow에서는 주기적을 메타스토어로 heartbeat를 전송하여 완료여부, 종료여부 등을 판단한다
- 테스크가 실패하여도 Airflow 시스템 전체가 다운되는 것을 막기 위하여 테스크를 실행할 때는 새로운 프로세스를 만들거나 풀에서 가져와서 스케쥴러와 관련 없는 프로세스가 담당한다
- airflow에서는 **Executor** 유형에 따라 다양한 설치 환경 제공.

#### SequentialExecutor

- local에서 테스트 용으로 사용되는 익스큐터.
- 테스크를 순차적으로 한 번에 하나씩만 할 수 있음

```
          |            |                           |
webserver -> Database <- Scheduler -> [subprocess] -> DAG directory
          |            |                           |
```

#### LocalExecutor

- 단일 호스트에서 테스크를 병렬로 실행할 수 있음(FIFO)
- SequentialExecutor와 달리 테스크 처리를 위한 하위 프로세스가 여러 개 존재한다(
- AIRFLOW__CORE__DAG__CONCURRENCY, AIRFLOW__CORE__MAX_ACTIVE_RUNS_PER_DAG로 테스트 수 조절 하거나 AIRFLOW__CORE__PARALLEISM으로 하위 프로세스 수 조절


```
          |            |                           |
webserver -> Database <- Scheduler -> [subprocess1] -> DAG directory
          |            |           -> [subprocess2]|
          |            |                           |
```


#### CeleryExecutor

- 분산환경에서 사용.
- 내부적으로 **Celer**를 사용. 태스크들은 대기열에 등록되고 각 worker들이 등록된 테스크를 가져와서 개별 처리
- Celery의 대기 broker(queue)로는 RabbitMQ, Redis 등이 사용.
- celery worker가 스케쥴러의 task executor의 역할을 담당한다. 이 때 Celery worker는 DAG directory와 DB에 접근할 수 있어야 한다(ex. AIRFLOW__CELERY_RESULT_BACKEND=db+mysql://user:password@10.10.10.10/airflow). worker는 *airflow celery worker* 명령어로 설치
- **Flower**라는 모니터링 도구도 제공


```
          |            |           |        |
webserver -> Database <- Scheduler -> Queue <- Celeery Worker 1 -> [subprocess1, subprocess2 ...]
          |            |           |--------|X---------------------------------------------------
          |            |          -> DAGs   <- Celeery Worker 2 -> [subprocess1, subprocess2 ...]
```



#### KubernetesExecutor

- 쿠버네티스에서 실행되는 익스큐터. Airflow task들을 배포하기 위해 k8s API와 통합됨
- 모든 task들은 k8s의 pod에서 실행된다. 하나의 task는 하나의 pod에서 실행된다. task가 실행될때마다 pod가 생성.
- helm 차트를 이용하여 배포하는 경우가 많음.
- pod에서 DAG 파일에 접근할 수 있도록 하기 위해서는 3가지 방법 중 선택해야 한다
  1. PersistentVolume 이용
  2. Git-sync init container를 사용해서 repository의 최신 DAG 가져오기
  3. docker image 빌드 시 DAG 포함

- - -

Metadata DB(metastore)
===========

- webserver, scheduler, executor가 상태 및 configuration을 저장하는 DB
- SQLAlchemy python ORM framework를 사용.

- -


DAG directory
============

- scheduler와 executor가 읽는 file system

- - -
