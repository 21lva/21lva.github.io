---
layout: post
title:  "Batch"
name: sb_batch.md
category: spring-boot
---

- batch : 일괄 처리. 단발성으로 대용량의 데이터를 처리
- Batch application의 조건
  - 대용량 데이터
  - 자동화
  - 잘못된 데이터를 충돌/ 중단 없이 처리
  - 로깅 등을 이용하여 잘못된 점을 추적
  - 지정된 시간안에 처리.
  - 다른 어플리케이션에 방해 X


- 주의 사항
  - 단순한 구조와 로직 사용
  - 데이터 무결성 유지하는 유효성 검사 등의 방어책
  - System IO 사용을 최소화. 많은 IO는 DB와의 연결을 늘려서 성능에 문제가 될 수 있다.
  - 다른 어플리케이션에 영향을 주는지 확인
  - 스케쥴러를 위해 [Quartz](http://www.quartz-scheduler.org/)같은 프레임워크를 사용해야 한다

- 배치 처리(읽기->처리->쓰기 흐름)
  - Read : 데이터 저장소에서 특정 데이터를 읽음
  - Processing : 원하는 방식으로 데이터를 가공/처리
  - Write : 수정된 데이터를 저장소에 쓰기

- - -

배치 처리
===============

배치 처리 관계도
---------------

```
JobLauncher - Job - Step - item - ItemReader
      |        |      |         - ItemProcessor
      [JobRepository]           - ItemWrite
```

- Job과 Step은 1:M 관계 : Job은 여러 개의 Step을 만들어서 일처리한다

Job
----

- 배치 처리 과정을 하나의 객체로 만든 것.
- 여러 개의 Step(Bean)으로 구성된 **컨테이너** 이다

- 생성을 위해서는 **JobBuilderFactory** 혹은 **JobBuilderFactory.get** 을 이용한다

```java
@Autowired
private JobBuilderFactory jobBuilderFactory;

@Bean
public Job simpleJob(){
  return jobBuilderFactory.get("simpleJob")
                          .start(simpleStep())
                          .build();
}
```

- get : builder 생성. JobBuilderFactory 객체를 생성할 때 전달 받은 JobRepository를 repository로 이용하여 생성
- start : 특정 builder를 생성. 상황에 따라 Job 생성 방법이 다르기 때문에 이런 식으로 간접적으로 builder 생성
- build : Job을 생성

JobInstatnce
-------------

- Batch에서 Job이 실행될 때 사용되는 Job의 실행 단위
- 특정 시점에 JobExecution이 실패를 하게 되면 새로운 JobInstance가 만들어지는 것이 아닌 실패한 JobInstance가 다시 JobExecution을 실행

JobExecution
--------------

- JobInstance에 대해 한 번 시행(성공)되는 객체
- 멤버변수
  - jobParamters : 실행에 필요한 매개변수
  - jobInstance : 실행의 단위가 되는 객체
  - stepExecution: StepExecution을 여러 개 가질 수 있는 collection
  - status : Job의 실행 상태. COMPLETED, STARTING(기본값), STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN
  - startTime : 실행된 시간. 시작하지 않았을 시 null
  - createTime : JobExeuction 객체의 생성 시간
  - endTime : JobExecution 이 끝난 시간
  - lastUpdated : 마지막으로 수정된 시간
  - exitStatus : Job의 실행에 대한 결과. UNKOWN(기본값), EXECUTING, COMPLETED, NOOP, FAILED, STOPPED
  - executionContext : Job 실행시 유지되어야 하는 user 데이타
  - failureExceptions : Job 실행 중 에러를 저장하는 List
  - jobConfigurationName : Job 설정 이름

JobParameters
-------------

- Job 실행에 필요한 정보를 key-value(Map) 형식으로 저장
- JobInstance를 구별하는 기준
