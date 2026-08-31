## 클릭하우스 개요 
- 클릭하우스는 온라인 분석처리(OLAP)를 위한 고성능 열 기반 SQL 데이터베이스 관리 시스템(DBMS)입니다. 오픈 소스 소프트웨어와 클라우드 서비스로 제공 된다. 

많은 사용 사례에서 분석 쿼리는 "**실시간**"이어야 한다. (즉, 1초 이내에 결과가 반환되어야 한다.)

## 행 기반(Row-oriented) VS 열 기반(Column-oriented)

|구분|DBMS|특징|
|------|---|---|
|열 기반|ClickHouse|고성능 OLAP DB, 분산/병렬 처리 우수|
|열 기반|Amazon Redshift|AWS 기반 DWH, PostgreSQL 유사|
|열 기반|Google BigQuery|서버리스, 쿼리 요금제 기반|
|행 기반|MySQL|대표적인 오픈소스 RDBMS|
|행 기반|PostgreSQL|확장성과 정합성 우수|
|행 기반|MariaDB|MySQL 포크, 라이센스 자유|
|행 기반|Oracle DB|기업용 상용 DB|
|행 기반|Microsoft SQL Server|MS 생태계에 최적화|
|행 기반|SQLite|임베디드 환경, 가벼운 단일 파일 DB|
|혼합구조|Apache Cassandra|분산형 NoSQL, 열 기반 구조지만 row-like 접근 방식|

**--> OTLP 방식은 CRUD에 적합, OLAP는 특정 칼럼 조회 및 대량 집계 통계에 적합함.**

## OLTP vs OLAP
- OLTP는 Online Transaction Processing의 약자
- OLAP는 Online Analytical Processing의 약자 

### OLTP vs OLAP 비교 테이블
|목적|추천 방식|대표 DB|
|------|---|---|
|OLTP (실시간 트랜잭션, CRUD)|행-기반|MySQL, PostgreSQL, Oracle|
|OLAP (분석, 집계, 리포트)|열-기반|ClickHouse, Redshift, BigQuery|
|시계열 데이터|ClickHouse|InfluxDB, TimescaleDB|


## 엔진 (Engine)
- 엔진은 **데이터베이스 엔진**과 **테이블 엔진**으로 나누어 진다.  

### 데이터베이스 엔진
클릭하우스는 기본적으로 구성가능한 테이블 엔진과 SQL언어를 제공하는 **Atomic** 데이터베이스 엔진을 사용한다. 

1. Atomic
- 이 Atomic엔진은 비차단 쿼리와 원자적 쿼리를 지원합니다 DROP TABLE. RENAME TABLE데이터베이스 EXCHANGE TABLES엔진 Atomic이 기본적으로 사용됩니다.

2. Backup
- 읽기 전용 모드에서 백업에서 테이블/데이터베이스를 즉시 첨부할 수 있습니다.
3. Lazy
- 마지막 액세스 후 몇 초 동안 만 테이블을 RAM에 보관합니다 expiration_time_in_seconds. Log 유형 테이블에만 사용할 수 있습니다.
4. MaterializedPostgreSQL
- PostgreSQL 데이터베이스의 테이블을 사용하여 ClickHouse 데이터베이스를 만듭니다.
5. MySQL
- 원격 MySQL 서버의 데이터베이스에 연결하고 ClickHouse와 MySQL 간에 데이터를 교환하기 위한 쿼리를 INSERT수행 합니다.SELECT
6. PostgreSQL
- 원격 PostgreSQL 서버의 데이터베이스에 연결할 수 있습니다.
7. Replicated
- 이 엔진은 Atomic 엔진을 기반으로 합니다. ZooKeeper에 기록되고 지정된 데이터베이스의 모든 복제본에서 실행되는 DDL 로그를 통해 메타데이터 복제를 지원합니다.
8. SQLite
- SQLite 데이터베이스에 연결하고 ClickHouse와 SQLite 사이에서 데이터를 교환하기 위한 쿼리를 INSERT수행 할 수 있습니다.SELECT


### Atomic
Atomic 엔진은 비동기 DROP 테이블과 RENAME 테이블 쿼리, EXCHANGE 테이블 쿼리를 지원한다. 클릭하우스의 DEFAULT 데이터베이스 엔진 이다.

#### Table UUID
각각의 아토믹 엔진에서의 테이블은 UUID를 가진다.
`/clickhouse_path/store/xxx/xxxyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy/`
`xxxyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy` = UUID 이다.  

#### `RENAME` TABLE 
UUID를 수정하거나, 테이블 데이터를 이동시키지 않는다. 
이 쿼리는 즉각 실행되며, 다른 쿼리를 기다리지 않는다.

#### `DROP/DETACH` TABLE
`DROP TABLE` 사용 시, 데이터는 지워지지 않는다.
Atomic 엔진은 그저 마킹해놓는다. 
`/clickhouse_path/metadata_dropped/` 백그라운드 스레드에 알립니다.
`database_atomic_delay_before_drop_table_sec` 값의 세팅에 따라 테이블 삭제 delay를 정할 수 있다.   
`database_atomic_wait_for_drop_and_detach_synchronously` 값을 통해 동기화 모드를 정할 수 있다.  
이 경우엔, `DROP` 쿼리는 실행중인 `SELECT, INSERT` 쿼리 및 다른 쿼리들이 완료될 때 까지 기다립니다.  

#### `EXCHANGE` TABLE
EXHCHANGE 쿼리는 테이블을 바꾼다.
아토믹 엔진이 아닌 경우 아래와 같은 명령어를 사용한다.
```
Non-atomic

RENAME TABLE new_table TO tmp, old_table TO new_table, tmp TO old_table;
```
아토믹 엔진인 경우 아래 명령어를 사용한다.
```
Atomic

EXCAHNGE TABLES new_table AND old_table;
```

#### ReplicatedMergeTree in Atomic Database
Atomic 데이터베이스 엔진에서 ReplicatedMergeTree 테이블 엔진을 사용하는 경우, Zookeeper와 replica name에 대한 엔진파라미터를 **안 쓰는 것이 권장된다**.
이 경우엔 `default_replica_path`와 `default_replica_name`이 사용된다.
만약 엔진 파라미터를 사용하고 싶다면 `uuid`정도를 사용하면 된다. 이 파라미터는 각각의 테이블에 있는 Zookeeper에 대해서 유니크한 경로를 생성하는 것을 보장한다. 

#### Metadata disk
`disk`가 `SETTINGS`에 명시되어 있다면, 테이블 메타데이터 파일을 저장하기 위해 사용된다.  
`CREATE TABLE db (n UInt64) ENGINE = Atomic SETTINGS disk=disk(type=='local', path='/var/lib/clickhouse-disks/db_disk')`
명시되어 있지 않다면 disk는 `database_disk.disk`가 **default**로 사용된다.


### 테이블 엔진

|엔진|설명|
|------|------|
|MergeTree|[기본형] 정렬된 칼럼 기반 저장(대용량 데이터 최적화)|
|ReplacingMergeTree|같은 PK에 대해 새로운 값으로 자동 대체|
|SummingMergeTree|같은 PK의 데이터를 자동 집계(합산)|
|AggregatingMergeTree|미리 집계된 데이터를 효율적으로 저장|
|VersionedCollapsingMergeTree|버전 필드를 기반으로 중복 제거|
|CollapsingMergeTree|상태값(1, -1)을 기준으로 행을 제거|
|GraphiteMergeTree|Graphite 시간 시계열 데이터용 엔진|
