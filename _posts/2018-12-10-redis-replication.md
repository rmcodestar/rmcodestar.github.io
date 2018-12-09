---
layout: post
title: Redis replication
category: REDIS
tag: [REDIS,Redis replication]
---
# Redis replication
서비스의 지속성을 위해 레디스는 복제 방식을 제공한다. 
레디스 서버를 마스터, 슬래이브로 구성하여 마스터 서버 다운 시에도 애플리케이션은 슬래이브로 접속해서 서비스를 계속할 수 있다.

<br/>

## replication이란?
복제란 레디스의 데이터를 거의 실시간으로 다른 레디스 노드에 복사하는 작업이다.
따라서 서비스를 제공하던 첫 번째 레디스 노드(마스터)가 다운되더라도, 데이터를 받은 두 번째 레디스 노드(슬레이브)가 서비스를 계속 할 수 있다.

<br/>

## 복제 구성하기
1. slave 서버에 관련 레디스 설정을 해준다
2. 마스터 노드를 시작한다.
3. 슬레이브 노드를 시작한다.

<br/>

## 슬레이브 노드의 레디스 설정
### master 정보 입력
```
slaveof <masterip> <masterport>
```

<br/>

### 슬레이브 우선순위
슬레이브 노드가 여러대 있으면 어느 슬레이브가 마스터가 될지 우선순위를 정하는 파라미터이다.   

```
slave-priority 100
```
* 기본값은 100이고, 숫자가 적을수록 우선순위가 높다
* 0은 특별한 값으로 마스터가 되지 않도록 하는 값이다(여러개의 슬레이브가 있을 때만 적용)

<br/>


### 선택) 비밀번호 설정
master에 비밀번호 설정이 되어있다면 슬레이브도 설정해주어야한다.

```requirepass <password>```

```masterauth <password>```

<br/>

### 시나리오
마스터에 아무런 파일이 없는 상태로 다시 서버가 시작되었다.
그러면 슬레이브 노드의 데이터는 유실될까?

> 그렇다. 마스터 데이터의 전체 동기화하기 때문에 슬레이브의 메모리에 있던 데이터도 삭제된다.


<br/>



## 모니터링

#### 모니터링 시나리오 구성 ####

로컬에는 총 4개의 레디스 인스턴스가 존재한다.

* 마스터(`M1`)에는 2개의 1차 슬레이브가 존재(`S1`, `S2`)

* 1차 슬레이브인 `S1`에게는 2차 슬레이브 `S1-1`가 존재

```
            +--------+
            |   M1   |
            | (6382) |
            +--------+
                |
  +--------+    |    +--------+
  |   S1   |----+----|   S2   |
  | (6383) |         | (6385) |
  +---+----+         +--------+
      |         
  +---+----+       
  |  S1-1  |
  | (6384) |       
  +--------+       
```

마스터 인스턴스에서 2차 슬레이브의 정보도 확인할 수 있을까?



master instance(`M1`)에서

```
127.0.0.1:6382> info replication
role:master
connected_slaves:2 
slave0:ip=127.0.0.1,port=6383,state=online,offset=1508,lag=0 
slave1:ip=127.0.0.1,port=6385,state=online,offset=1508,lag=1
```

master에서는 1차 슬레이브의 정보만 보인다.



Slave instance (1차 : `S1`)에서

```
127.0.0.1:6383> info replication
role:slave
master_host:127.0.0.1 
master_port:6382
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0
slave_repl_offset:1690
slave_priority:100
slave_read_only:1
connected_slaves:1 #2차 슬레이브 정보
slave0:ip=127.0.0.1,port=6384,state=online,offset=1662,lag=1
master_repl_offset:1662
```

1차 슬레이브에서는 2차 슬레이브가 보인다.



slave instance(2차 : `S1-1`)에서

```
127.0.0.1:6384> info replication
role:slave 
master_host:127.0.0.1
master_port:6383
master_link_status:up
master_last_io_seconds_ago:6
master_sync_in_progress:0
slave_repl_offset:1718
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
```

2차 슬레이브의 마스터는 1차슬레이브인 `S1`이다.



<br/>


## reference
http://redisgate.com/redis/configuration/replication.php
