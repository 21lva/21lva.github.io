
# PostgreSQL Lock & Transaction

- PostGreSQL은 동시성 제어를 위한 여러 기능을 제공.
- 내부적으로는 MVCC를 통해서 데이터 일관성이 유지된다. 
- transaction 전체 isolation을 원하지 않는 application을 위해서 table-level lock, row-level lock 등을 제공한다. 성능적 이유로는 MVCC를 사용하는 것이 나은 선택이다. 

- 비관적(Pessimistic) 동시성 제어: 사용자들이 같은 데이터를 동시에 수정할 것이라고 가정. READ시에 lock을 설정하고 transaction이 commit되면 lock을 해제. wait을 설정해서 대기중인 사용자가 일정 시간만큼만 대기해서 무한정 대기하는 문제(starvation)을 방지

- 낙관적(Optimistic) 동시성 제어: 사용자들이 같은 데이터 수정을 하지 않을 것이라고 가정. READ 시점이 아닌 WRITE 시점에 동시 수정되었는지를 확인한다. *Version*을 관리해서 WRITE할때마다 Version 값을 1씩 올린다. 동시 수정이 된 경우에는 Version이 다르므로 에러를 발생시킨다. READ한 후에 데이터가 변경되어도 로직은 계속 수행되기 때문에 데이터 정합성 문제가 생길 수 있다.

- 다중 버전 동시성 제어(Multi-Version Concurrency Control, MVCC): 데이터에 접근하면 해당 시점 DB의 snapshot을 READ. 사용자가 DB를 업데이트하면 새로운 데이터와 기존 데이터를 비교해서 변경된 내역을 UNDO에 생성. 하나의 데이터에 여러 버전의 데이터가 존재하게 되고 사용자는 최근 버전의 데이터를 READ한다. lock을 사용하지 않기 때문에 기존의 RDBMS보다 빠르고 데이터를 읽을 때 다른 사용자가 해당 데이터를 수정해도 영향을 받지 않는다. 데이터에 여러 버전이 존재하기 때문에 주기적으로 데이터를 정리해야 한다. 어플리케이션에서 여러 버전의 데이터 중 어떤 버전을 사용해야하는지 등을 관리해야 한다.

- - -

## Transaction

- Transaction isolation level은 4가지이다: Serilzable, read commited, repeatable read, read uncommitted.
	- Serializable: 동시적으로 실행되는 transaction들이 순차적으로 하나씩 동작하더라도 같은 결과를 보여줌을 보장한다.
	- 다른 3가지 레벨은 몇개의 현상을 보여준다. Serializable에서는 이 4개 현상이 일어나지 않는다
- 현상
	- dirty read: transaction이 다른 transaction이 update한 uncommitted 데이터를 READ 가능
	- nonrepeatable read: transaction이 두 번 이상 데이터를 읽을때 그 사이에 다른 transaction이 데이터를 수정해서 commit하는 경우가 허용
	- phantom read: transaction이 같은 query 문을 중복 시행할 때 다른 transaction의 commit때문에 쿼리 결과가 달라지는 현상
	- serialization anomaly: transaction 그룹의 결과가 시행 순서에 따라 달라지는 경우.
![](att/Pasted%20image%2020230510162138.png)
- PostgreSQL에서 read uncommitted는 read commited와 같다. PostgreSQL의 MVCC를 위해서. 

### Read committed

- PostgreSQL의 default transaction isolation level.
- select문은 commit된 data만 볼 수 있다. 물론 다른 transaction이 아닌 transaction내부에서 write,update한 경우에는 데이터의 변경을 볼 수 있다.
- nonrepeatble read가 가능
- transaction이 수정하려하는데 다른 transaction이 이미 수정하는 경우에는 일단 대기한다
	- 전 transaction이 롤백되는 경우: 다음 transaction이 update
	- 전 transaction이 commit한 경우: 삭제된 경우에는 data를 무시하고, 수정된 경우에는 수정된 데이터에 다음 transaction 수정을 시도한다

- - - 

## Lock

- PostgreSQL에서는 동시성 제어를 위해서 **lock**과 **Transaction**을 지원
	- 배타적(exclusive) lock: 데이터 변경을 하고자할 때 transaction 완료시 까지 테이블 or row를 다른 트랜잭션이 READ하는 것을 방지
	- 공유(shared) lock: read lock이라고도 함. 데이터를 읽을때는 허용. 수정은 방지. 
- 대부분의 명령은 자동으로 적절한 lock을 획득함. 

### Table lock
![](att/Pasted%20image%2020230510154342.png)
- lock은 하나의 트랜잭션에서 유지. 
- savepoint를 설정하면 lock을 획득하고 savepoint가 롤백될때 lock도 반환.
- 아래의 lock들은 명령에 따라 자동으로 획득되어진다. 명시적으로 lock을 걸고 싶으면 *Lock*을 사용한다.
- lock 충돌은 다른 transaction내에서 이루어진다. 하나의 transaction에서는 lock간의 충돌이 이루어지지 않는다. 

#### Access Share
- access exclusive lock과 충돌 나머지는 충돌 하지 않는다
- select을 통해서 대상 테이블에 access share lock을 얻는다. 

#### Row share

- exclusive, access exclusive lock과 충돌
- select for update, select for share을 통해서 획득

#### Row exclusive

- share, share row exclusive, exclusive, access exclusive와 충돌
- update, delete, insert시에 획득(일반적으로 테이블 내 데이터를 수정하는 모든 명령에 대해서 동작)

#### Share update exclusive

- share update exclusive, share, share row exclusive, exclusive, access exclusive와 충돌
- vacum, create index concurrently, alter table validate, alter table을 통해서 획득
- 스키마 변경, vaccum 실행으로투버 테이블을 보호

#### Share

- row exclusive, share update exclusive, share row exclusive, exclusive, access exclusive와 충돌
- create index를 통해서 획득

#### share row exclusive

- row exclusive, share update exclusive, share, share row exclusive, exclusive, access exclusive와 충돌
- create trigger, 다양한 형태의 alter table 등을 통해서 획득
- 동시 데이터 변경으로투버 테이블을 보호하고 하나의 세션만 테이블을 차지하도록 보장

#### exclusive

- row share, row exclusive, share upate exclusive, share, share row exclusive, exclusive, access exclusive와 충돌. access share lock만 허용. 
- refresh materialized view concurrently를 통해서 획득

#### access exclusive 

- 모든 lock과 충돌
- drop table, truncate, reindex, cluster, vacuum full, refresh materialized view(concurrently X)에 의해 획득.
- lock 모드를 지정하지 않으면 기본적으로 적용되는 lock

### Row lock

![](att/Pasted%20image%2020230510155208.png)
- 하나의 row에대해서 lock을 거는 방식. 
- table lock과 마찬가지로 transaction이 종료되거나 savepoint가 rollback될 때 해제
- table lock과 마찬가지로 하나의 transaction혹은 subtransaction에서는 lock간의 충돌이 없다. 다른 transaction의 lock과는 충돌 가능.
- row lock은 select과 같은 query시에는 걸리지 않는다. 

#### for update

- select할때 검색된 row에 update를 위한 lock을 건다. 현재 row가 수정 및 삭제되는 것을 방지한다. 
- 해당 row에대한 update, delete, select for update, select for no key update, select for share, select for key share를 차단
- row에 대한 delete, 특정 column을 update할때 획득

#### for no key update

- 해당 row에대한 update, delete, select for update, select for no key update, select for share를 차단
- for update lock을 얻지 않는 모든 update에 대해서 획득 가능

#### for share

- update, delete, select for update, select for no key update를 차단

#### for key share

- 해당 row에대한 update, delete, select for update를 차단

- - -

## Deadlock

- explicit lock을 사용하면 **Deadlock**의 위험성이 생긴다.
	- 데드락: 두 개 이상의 transaction이 각자의 lock을 소유하고 다른 transaction의 lock을 기다리는 문제.
- PostgreSQL에서는 데드락 발생 시 자동으로 하나의 transaction을 *abort*하면서 데드락을 해결한다. 
- 데드락을 피하는 좋은 방법은 어플리케이션 레벨에서 여러 object(table, row, page)에 대한 lock을 걸때의 순서를 고려하는 것이다. 예를 들면, update하는 순서를 같게한다든가 하는 방식이다. 
- 데드락이 발견되지 않는다면 table, row lock을 원하는 transaction은 무기한 대기할 것이다. 하나의 트랜잭션을 오래 유지하는 것은 좋은 선택이 아니다.


- - -

## Reference

- [PostgreSQL: Documentation: 15: 13.3. Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html)
- [[PostgreSQL] DB Lock :: 불곰 (tistory.com)](https://brownbears.tistory.com/543)