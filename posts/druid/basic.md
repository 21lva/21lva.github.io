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
- segment를 특정 Data server(historical을 포함하는 서버)에 할당하고 historical process의 segment를 balancing하는 역할
	- 정확히는 historical process에게 segment을 load하거나 drop하도록 명령한다. 
	- load는 새로운 segment, replicated되어야하는 segment, 다른 historical에서 온 segment에게 수행된다
	- drop은 다른 historical로 가야하는 segment, 오래된 segment인 경우에 수행된다
- coordinator는 주기적으로 혹은 특정 시간에 cluster의 현재 상태를 판단하고 적절한 작업을 결정한다. 
	- 현재 상태 판단은 zookeeper에 저장되어 있다. 
	- used segment(load 되어야하는 segment)와 loading 규칙에 대한 정보는 metadata db에 저장되어있고 coordinator는 여기에 연결되어 있다.
- segment가 어떤 historical에 할당되어야하는지를 결정하는 과정
	1. 규칙에 따라서 historical process들을 정렬. segment는 process들간의 balance를 위해서 가장 적은 "capacity"를 보유한 process에 할당된다
	2. coordinator는 historical과 직접 통신하지 않고 zookeeper에 ephermeral node를 생성해서 historical process에 segment 할당을 명령
	3. 해당하는 historical process가 segment를 deep storage로부터 load
- coordinator는 db에서 "used segment"에 대한 정보와 실제 historical들에 올라가있는 segment 정보를 비교해서 used segment가 아닌 segment을 unload하라고 명령한다
- segment 중 오래된 segment는 unused로 표시된다. 다음 주기에서 coordinator는 해당 segment를 unload하라고 명령한다

- historical process가 restart하거나 사용 불가능하면 coordinator는 해당 historical에 속하는 segment를 drop되었다고 간주한다. 
- 충분한 시간 이후에는 segment는 다른 historical에 할당될 수 있다. 그러나 즉시 일어나지는 않는다. 짧은 시간안에 historical process가 다시 올라오면 segment들은 rebalancing되지 않는다.

- coordinator는 historical들의 utilization을 비교하고 차이가 threshold 이상이면 몇개의 segment들을 rebalancing한다. 
	- 최대 몇 개의 segment가 옮겨질지는 config로 설정 가능하다

- coordinator들은 segment를 compaction하거나 분리하는 역할도 수행한다.

#### Overload

- MiddleManager process(Data server에 존재)를 감시
- druid system의 ingestion을 조율한다
	- task를 전달받아서 분배하고 task 전후로 lock을 다루는 역할 수행
- middle manager에 ingestion을 할당하고 segment 배포(publish)를 조율한다.
	- publish: segment가 생성된 후에 deep storage에 저장하는 과정. metadata에 segment 정보가 생성되면 publish 된 것
- task가 특정 middlemanager에서 지속적으로(threshold 이상) 실패하면 *blacklist*에 등록한다. 최대 등록할 수 있는 blacklist는 전체의 20%이다. 

### Query server

- 사용자의 query를 data server나 다른 query server에 routing하는 기능 제공
- **Broker, Router**로 구성

#### Broker process

- 외부 사용자로부터 query를 전달받아서 data server에 포워딩 하는 역할.
	- 대부분의 query는 time 기간이 있음. segment는 time에 따라서 나눠져 있다. 
	- zookeeper의 metadata를 읽어서 segment들이 어떤 historical에 존재하는지를 확인(부팅시 historical은 zookeeper에 자기가 가지고 있는 segment들에 대해서 통보)
	- broker는 segment들의 timeline을 생성하고 각 timeline들을 어떤 segment들에 할당할지 결정
	- segment에 맞게 query를 subquery로 나눠서 각 historical에 요청. 
- sub query로부터 결과를 받으면 query 요청자에게 결과를 전달하기 전에 결과를 merge
- 보통 사용자는 query를 직접 data server로 보내기보다는 broker에게 보낸다

- broker는 **LRU cache**를 가지고 있음. cache는 local에 저장할 수도 있고 memcached같은 외부 cache 서비스에 저장할 수도 있다.
- broker는 cache에 segment 단위로 query 결과를 저장. 
- cache에 결과가 저장되어 있지 않은 경우에는 historical process에 query를 전달. historical에서 결과를 응답하면 broker는 해당 결과른 cache에 저장. 
- real-time데이터는 cache되지 않고 historical에 질의한다.


#### Router process

- optional
- druid의 broker, overload, coordinator의 통합 API gateway를 제공
- web console을 제공
	- web console: datasource, segment, task, data  process등 여러 정보를 확인 및 조정하는 기능을 제공하는 UI

### Data server

- job은 ingestion하고 query 가능한 데이터를 저장하는 역할
- **MiddleManager, Historical**로 나뉜다

#### Historical process

- storage & quering historical data를 관리하는 역할
- segment을 deep storage로부터 다운받고 해당 segment에 대한 query를 수행하고 결과를 전달한다. 
	- segment는 *druid.segmentCache.locations*에 지정된 장소(segment cache)에 저장한다. 
- coordinator는 historical에 segment를 할당하고 historical들의 segment에 균형을 맞춘다. historical은 자기들끼리 통신하지 않는다
- historical은 coordinator와 "직접" 통신하지 않고 zookeeper의 **ephermeral node**를 이용하여 상호작용한다. 
- hitorical process가 zookeepr의 새로운 node가 생성되면 해당 segment가 cache에 존재하는지 확인한다. 
	- 없으면, zookeeper에서 segment에 대한 meta data(segment의 deep storage 내의 위치, decompress하는 방법 등)를 획득한다. 해당 정보를 바탕으로 deep storage에서 데이터를 다운 받는다. 다운이 완료되면 zookeepr에 해당 정보를 쿼리할 수 있다는 정보를 공지한다. 
- segment cache는 **memory mapping**을 사용한다. 
	- cache는 memory의 일부를 소유하고 있는다. historical은 해당 영역에 segment의 일부를 load해서 query의 성능을 향상시킨다
	- query시 memory에 필요로 하는 segment 파일이 등록되어 있으면 historical은 해당 segment의 데이터를 메모리에서 직접 가져온다. 
	- query시 memory에 필요로 하는 segment의 일부가 없으면 disk에서 누락된 segement의 일부를 가져온다. 이 경우에는 다른 segment data가 메모리로부터 사라질 수 있다. 
	- OS의 사용 가능한 memory의 양이 *druid.server.maxSize*에 가깝거나 클 수록 메모리에 많은 segment를 load할 수 있기 때문에 query 성능을 향상시킬 수 있다
- write 작업은 허용되지 않는다

#### MiddleManager process

- 새로운 데이터에대한 ingestion을 진행. 
- 외부 데이터 소스에서 데이터를 읽고 새로운 segement를 만드는 역할
- **Peon process**: MiddleManger에 의해 생성되는 task execution 엔진 프로세스. 하나의 peon process당 별개의 jvm에서 실행된다. 모든 peon process는 자신을 생성한 MiddleManager와 같은 host에서 실행된다.

### Colocation

- Master/Data/Query server는 각각 위에 설명한 것처럼 process를 여러개 가질 수 있다. colocation을 하면 hardware 자원을 효율적으로 사용할 수 있다.
- 그러나 큰 cluster라면 해당 프로세스들을 각각의 서버에 분리해야할 필요가 있다. 

#### Coordinators & Overloads

- coordiantor는 segment를 관리하도록 historical을 감시하는 역할, overload는 ingestion을 조율하는 역할. 
- segment의 수가 증가할 수록 coordinator의 필요 수와 overload의 필요 수의 차이가 커진다. 
- segment의 수가 많은 클러스터의 경우에는 coordinator와 overload를 분리해서 coordinator의 수를 늘려야 한다.

#### Historical & MiddelManager

- ingestion 혹은 query의 부하가 많아질 때 historical과 MiddleManager 프로세스들을 분리할 필요성을 느낄 것이다.
- 특히, historical은 memory에 segment를 올리기 위해서 많은 메모리를 필요로하기 때문에 둘을 분리하는 것이 효율적일 수도 있다.


### 외부 의존성

- druid는 3개의 외부 의존성을 갖는다: deep storage, metadata db, zookeeper

#### deep storage

- druid는 deep storage에 ingestion된 데이터를 저장한다.
- deep storage는 모든 druid server로부터 접근 가능한 공유 file storage. e.g. s3, hdfs.
- druid에서 deep storage의 용도는 데이터를 backup해놓는 것이다. 
- druid는 데이터를 **segment** 단위로 저장. historical은 segmenet를 자신의 cache 디렉토리에 저장해놓고 사용. 일반적으로 query시에는 deep storage에 접근하지 않는다. 

#### metadata db

- segment usage, task 정보와 같은 다양한 shared system metadata를 저장해 놓는 db.
- cluster 환경에서는 RDBMS(postgresql, mysql) 사용. 
- metadata db에 저장하는 정보
	- segment: segment관련 정보. coordinator에서 사용. used와 같은 정보 포함
	- rule: segment가 위치해야하는 장소에 대한 규칙. coordinator에서 사용
	- configuration
	- task-related: overload와 middelmanager가 task 관련 정보로 사용
	- audit

#### zookeeper

- 현재 cluster의 상태 관리용
- zk를 사용하는 경우
	- leader selection(coordinator, overload)
	- segment를 publish하는 protocol
	- coordinator와 historical이 segment의 load/drop을 결정
	- overload와 middelmanager의 task 관리

- - -

## Storage

### Datasources & Segments

![](att/Pasted%20image%2020230528164533.png)

- druid는 **Datasource**라는 곳에 data를 저장.
- datasource는 RDBMS와 비슷한 table 구조. 
- 기본적으로 time으로 paritioning되어 있고 추가로 설정된 기준으로도 파티셔닝.
	- 각 time 범위는 **chunk**라고 부른다. 각 chunk 단위에서는 segment들은 복수개 존재 가능.
- segment들은 하나의 file. 수백만개의 rows를 포함. 
- segment들은 middlemanager에 의해 생성되며 이때 상태는 *mutable, uncommitted*. uncomitted segment들은 query 가능
- segment들은 추후에 요청되는 query들의 응답성을 높이기 위해서 data파일을 *compact, index*한다.
	- column구조로 변경
	- bitmap index 적용
	- 압축
- 주기적으로 uncomitted segment들은 deep storage로 *publish*되고 이후에는 *immutable, committed* segment가 된다. 
- *immutable, committed* segment들은 middlemanager에서 historical로 옮겨진다. 

- metadata에는 segment의 크기, schema, deep storage 위치 등을 저장하는 정보가 저장되어 있다. coordinator는 해당 정보를 이용하여 data가 cluster에서 사용가능한지 확인한다. 

### Indexing & handoff

- indexing: 새로운 segment가 생성될때 동작
- handoff: *publish*되고 historical process에서 사용가능할때 동작

- Indexing 동작 과정:
	- indexing task는 새로운 segment 생성을 시작. 시작 전에 segment의 *identifier*를 결정. 
	- indexing task가 append이면, overload의 allocate api를 호출해서 이미 존재하는 segment들에 partition을 추가한다. 
	- append 작업이 아닌 경우(overwritting)이면, interval에 lock을 걸고 segment들의 새로운 version을 만든다
	- indexing task가 real-time인 경우에는 segment는 지금 즉시 query 가능하다. 그렇지만 publish되지 않은 상태
	- indexing task의 작업이 끝났다면 deep storage에 데이터를 저장한다. 그후에 metadata db을 수정해서 segment를 publish 한다. 
	- indexing task가 real-time task이면 historical process가 segment를 load할때까지 기다린다. real-time task가 아니면 해당 task는 즉시 종료된다. 
- Coordinator/ historical side(handoff):
	- coordinator는 주기적으로 metadata db에서 새로 publish된 데이터가 있는지 확인
	- 새롭게 publish되었고 used인 segment가 존재하고 해당 segment가 사용 가능(historical에 load된 상태)하지 않으면, coordinator는 load 작업을 진행한 segment를 결정한다. 그후에 zk를 수정해서 historical process가 load하도록 명령한다.
	- historical이 segment를 load한다. 만약 ingestion task가 real-time이면 이때 task가 종료된다. 

### Identifier

- segment의 identifier는 4개로 구성된다
	- datasource 이름
	- time interval
	- version 
	- partition number
 
```
//예시
`clarity-cloud0_2018-05-21T16:00:00.000Z_2018-05-21T17:00:00.000Z_2018-05-21T15:56:09.909Z_1`
```


### segment lifecycle

- segment는 3개의 영역에 걸쳐서 생명주기를 갖는다
- 


- - -

## Segment

- druid는 time을 기준으로 파티셔닝된 segment file로 데이터를 저장한다.
- 일정 time동안 데이터가 없으면 segment가 만들어지지 않는다. 
- 일정 time(*segmentGranularity*)동안 여러개의 ingestion job에 의해서 data가 들어온 경우에는 여러 개의 segment들을 생성한다. 
	- Compaction: druid는 일정 time동안 만들어진 여러 segment들을 하나의 segment로 합치는 과정을 거친다. 
- druid에서는 segment 하나의 크기로 300MB~700MB를 권장한다. 하나의 segment가 너무 크면 partitioning 된다.

### structure

- segment 파일은 columnar: 각 column의 data들은 다른 data structure를 가진다. 
- column을 따로 저장하면 특정 column에 해당하는 query들의 성능을 향상시킬 수 있다. 
- 기본적으로 column의 type은 3가지: timestamp, dimensions, metrics
- timestamp, metric은 integer, float이고 *LZ4*로 압축된다. query가 요청되면 압출을 해제하고 해당되는 row를 획득한 후에 요청된 aggregation 작업을 수행한다. 
- dimension column은 *filter, group by*를 지원하기 위해 약간 다른 구조를 갖는다. dimension은 3가지 구조를 갖는다: dictionary, list, bitmap

### null 값

- druid string dimension column은 ''와 null을 같은 취급.
- 숫자 or metric clumn은 null을 사용할 수 없기 때문에 0을 사용. 
- 그러나 druid는 null을 다루기 위해서 SQL 호환성을 제공. 
	- string column에서 ''와 null을 구분
	- 숫자 column에서 0 대신 null 사용

### Sharding

- 하나의 datasource와 하나의 time interval에는 여러 segment 존재 가능.
	- block: 하나의 interval의 segment들의 오음
- 어떤 interval에 segment가 복수개 존재할 때, 해당 interval에 query가 들어오면 이 block이 전부 load되어야 가능하다. 
- Linear shard spec은 위 규칙에서 예외. 


- - -

## Reference

- [Druid 개발 배경 — Metatron User Manual 0.4.3 문서 (metatron-app.github.io)](https://metatron-app.github.io/metatron-doc-discovery/discovery/part01/druid_background.html)
- [Introduction to Apache Druid · Apache Druid](https://druid.apache.org/docs/latest/design/)