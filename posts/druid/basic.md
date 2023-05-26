# Druid

- druid에 대한 정보와 간략한 아키텍처, 구성 요소에 대해서 공부하고 정리한 글

- - -

## Apache Druid?

- 큰 사이즈의 데이터의 OLAP에 특화된 DB.
	- OLAP: 
	- OLTP: 
- real-time ingestion, fast query, high uptime이 특징,
- 데이터를 빠르게 slice & dice 할 수 있다.

### 드루이드 특징

- columnar storage format: column 기반 스토리지를 사용한다. column 기반으로 query할 수 있다. 이 방법은 몇 개의 column에 해당하는 query를 진행하는 경우에 높은 성능을 보여준다. 드루이드는 어그리게이션의 성능을 위해서 column의 데이터 타입에 맞게 최적화한다.
- real-time ingestion: 데이터가 발생하는 즉시 ingestion하여 query할 수 있게 한다. 드루이드는 batch 혹은 realtime으로 데이터를 처리한다.
- 병렬처리
- self-healing, self-balancing: 드루이드는 하나의 component가 영향을 받으면 스스로 데이터를 re-balancing한다. 하나의 서버가 다운되면 routing을 변경해서 새로운 서버가 동작할때까지 유지한다.
- Cloud native: ingestion이후에 데이터는 deep storage라는 곳에 복제된다. deep storage는 주로 cloud storage, hdfs, 공유 file system가 쓰인다. 드루이드에 문제가 생기고 재기동되면 deep storage에서 데이터를 불러와서 복구한다. 드루이드 전체가 아닌 일부에서 문제가 생기면 replication이 되어 있기 때문에 쿼리하는데에는 문제가 없다. 적은 downtime으로 HA를 보장한다.
- time-based partitioning: 드루이드는 가장 먼저 time을 기준으로 데이터를 partitioning한다. 사용자는 자기만의 기준으로 이차 partitioning을 설정할 수 있다. time-based query를 사용하면 해당하는 partition만 접근하기 때문에 성능적으로 효율적이다.

### 드루이드를 사용해야할 때

- insert는 자주 일어나지만 수정은 거의 없는 경우
- query의 대부분이 aggregation인 경우. 
- query 요구사항이 100ms 이하인 경우
- data가 time과 관련된 경우. 
- query가 하나의 큰 테이블에 해당되는 경우
- 높은 cardinality를 가지는 데이터인 경우. 
- kafka, hdfs, object storage 등에서 데이터를 읽어와야하는 경우

### 드루이드를 사용하지 말아야 할 때

- update가 자주 일어나고 update의 성능이 좋아야하는 경우.
- query의 latency가 중요하지 않은 경우
- 큰 테이블들에 대한 *join* 연산이 필요한 경우

- - -

## 아키텍처

![](att/Pasted%20image%2020230526211859.png)

- 드루이드는 cloud에 적합한 분산 구조 
- 드루이드는 **Coordinator, Overload, Broker, Router, Historical, MiddelManager**로 구성되어 있고, 외부시스템인 **Metadata Storage, Zookeeper, DeepStorage**에 의존하고 있다

### Master Server

- data ingestion과 avaliability(가용성)을 조율하는 역할
- 2개의 process로 구성: **Coordinator, Overload**

#### Coordinator 

- Historical process(Data server에 존재)을 감시.
- segment를 특정 Data server(historical을 포함하는 서버)에 할당하는 역할
- historical process의 segment를 balancing하는 역할

#### Overload

- MiddleManager process(Data server에 존재)를 감시
- druid system의 ingestion을 조율한다
- middle manager에 ingestion을 할당하고 segment 배포(publish)를 조율한다.

### Query server

- 

### 외부 의존성



- - -

## Reference

- [Druid 개발 배경 — Metatron User Manual 0.4.3 문서 (metatron-app.github.io)](https://metatron-app.github.io/metatron-doc-discovery/discovery/part01/druid_background.html)
- [Introduction to Apache Druid · Apache Druid](https://druid.apache.org/docs/latest/design/)