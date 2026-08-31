우선 기본적으로는 우리는 카산드라를 사용했는데, 그 이유는 
1 멀티리전 (확장성)
2 데이터 인풋이 많기 때문이고 (insert가 빠른가 봄)
3 데이터를 어떻게 빨리 뒤져보는가

이 부분이 중요했다고 한다.  
우리는 카산드라를 마이그레이션하는 작업을 할 것이다.  

테이블 엔진 선택 과정 및 클러스터 환경구성편 이다.

1 우리의 특성상 비디오/오디오 metadata를 저장할 테이블엔진 
2 비디오/오디오 레코더를 저장할 테이블 엔진

위의 2가지를 정해야 했다.

그리고 고려해야할 부분은 LIVE에 쓰이는 건지 VOD에 쓰이는건지
(우리는 VOD를 고려해서 사용함)

서비스의 특성상 항상 복제본을 들고 있으면 좋으므로,  

Replicated Merge Tree와 
metadata는 방송시작/종료 때 마다 업데이트가 되어서, Replicated ReplacingMergeTree 엔진을 쓴다. 중복된 데이터를 제거하거나 특정 기준에 따라 최신 데이터를 유지하고자 할 때 사용하는 엔진이다.
record는 단순 insert라 기본 머지트리 엔진을 사용한다.  


클러스터링 구성은 우리가 csdr1, csdr2, csdr3 호스트를 가지고 있으므로,
각각의 호스트에 클릭하우스 컨테이너 하나와 클릭하우스 키퍼 하나를 작동시키고 클러스터링 하는 것을 목표로 하였다.

### 클릭하우스 서버 클러스터 확인 방법 
```
clickhouse-01 :) SELECT * FROM system.clusters;



SELECT *
FROM system.clusters

Query id: 4b3f9daf-5405-4836-a150-b9defaec23e7

Row 1:
──────
cluster:                 default
shard_num:               1
shard_name:
shard_weight:            1
internal_replication:    0
replica_num:             1
host_name:               localhost
host_address:            127.0.0.1
port:                    9000
is_local:                1
user:                    default
default_database:
errors_count:            0
slowdowns_count:         0
estimated_recovery_time: 0
database_shard_name:
database_replica_name:
is_active:               ᴺᵁᴸᴸ
replication_lag:         ᴺᵁᴸᴸ
recovery_time:           ᴺᵁᴸᴸ

Row 2:
──────
cluster:                 recorder_cluster
shard_num:               1
shard_name:
shard_weight:            1
internal_replication:    0
replica_num:             1
host_name:               114.203.86.40
host_address:            114.203.86.40
port:                    9000
is_local:                1
user:                    default
default_database:
errors_count:            0
slowdowns_count:         0
estimated_recovery_time: 0
database_shard_name:
database_replica_name:
is_active:               ᴺᵁᴸᴸ
replication_lag:         ᴺᵁᴸᴸ
recovery_time:           ᴺᵁᴸᴸ

Row 3:
──────
cluster:                 recorder_cluster
shard_num:               2
shard_name:
shard_weight:            1
internal_replication:    0
replica_num:             1
host_name:               114.203.86.41
host_address:            114.203.86.41
port:                    9000
is_local:                0
user:                    default
default_database:
errors_count:            0
slowdowns_count:         0
estimated_recovery_time: 0
database_shard_name:
database_replica_name:
is_active:               ᴺᵁᴸᴸ
replication_lag:         ᴺᵁᴸᴸ
recovery_time:           ᴺᵁᴸᴸ

Row 4:
──────
cluster:                 recorder_cluster
shard_num:               3
shard_name:
shard_weight:            1
internal_replication:    0
replica_num:             1
host_name:               114.203.86.42
host_address:            114.203.86.42
port:                    9000
is_local:                0
user:                    default
default_database:
errors_count:            0
slowdowns_count:         0
estimated_recovery_time: 0
database_shard_name:
database_replica_name:
is_active:               ᴺᵁᴸᴸ
replication_lag:         ᴺᵁᴸᴸ
recovery_time:           ᴺᵁᴸᴸ

4 rows in set. Elapsed: 0.003 sec.
```


### 클릭하우스 키퍼 확인 방법
```
docker compose exec -it clickhouse-keeper3 /bin/bash
```

```
clickhouse-keeper-client -h localhost -p 9181 --connection-timeout 30 --session-timeout 30 --operation-timeout 30
```

```
$ stat 
===============================
ClickHouse Keeper version: v25.6.2.5-stable-51a12888a03cc2b211c90e16e4154761f43b889d
Clients:
 127.0.0.1:25826(recved=2,sent=2)
 114.203.86.41:36606(recved=7942,sent=7950)
 127.0.0.1:16744(recved=0,sent=0)
 114.203.86.42:5380(recved=7946,sent=7956)

Latency min/avg/max: 0/1/118
Received: 16419
Sent: 16438
Connections: 3
Outstanding: 0
Zxid: 0x15c4f
Mode: leader
Node count: 106
```

클러스터 구성하면서 어려웠던 점은 docker_compose.yml에 대한 이해와 
keeper_config.xml과 cluster.xml을 작성하는 부분에서 이해가 어려웠고, 각 호스트 설정 및 네트워크 설정에서 애먹었다.   
도커 네트워크를 network_mode=host로 쓸건지, 아님 네트워크를 브릿지로 해서 쓰냐에 따라서 구성방법이 또 달라지니...
그래도 여찌여찌 잘 된거 같아서 다행이다.