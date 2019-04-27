---
layout: post
title: Redis Persistence
category: REDIS
tag: [REDIS,Redis Persistence, AOF, RDB]
---

# Redis Persistence

데이터를 메인 메모리(RAM)에 보관하는 레디스는 데이터의 안전한 보관과 백업을 위해 디스크 쓰기 방식을 제공한다
레디스에서는 두가지 디스크 쓰기 방식을 지원한다

* RDB(snapshot) 
  * 파일은 특정한 간격마다 메모리에 있는 레디스 데이터 전체를 디스크에 쓴다
  * 특정 시점 마다 백업을 받을 때 사용
  * 바이너리 파일

* AOF(appendonlyfile)
  * 명령이 실행될때 마다 기록되는 파일
  * 텍스트파일



## AOF
* 입력/수정/삭제 명령이 실행될 때 마다 기록됨 (조회 명령은 기록되지 X)
* text파일이므로 편집이 가능하다.
* rewrite 기능이 있음


![AOF](https://raw.githubusercontent.com/rmcodestar/rmcodestar.github.io/master/public/img/20190427-aof.jpeg)

<br/>

**[복구시나리오]**  실수로 `FLUSHALL` 명령으로 메모리에 있는 데이터 전체를 날렸을 경우
> 즉시 레디스 서버를 shutdown하고, appendonly.aof 파일에서 `FLUSHALL` 명령을 제거 한 후 레디스를 다시 시작하면 데이터 손실없이 DB를 살릴 수 있다

<br/>

### appendonly.aof 파일 형식
* AOF는 명령 실행 순서대로 텍스트로 쓰여진다. 편집 가능하다. 
* `*`는 명령 시작을 나타낸다. 숫자는 명령과 인수의 개수이다. 
* `$`는 명령이나 인수, 데이터의 바이트 수이다. 한글은 UTF8로 했을 경우 한 글자에 3바이트이다.

```
*3
$3
SET
$8
user_aaa
$7
Charlie
*2
$4
INCR
$5
del-a
```

무슨 명령어를 쳤는지 추적해보자.
> SET user_aaa Charlie
>
> INCR del-a


<br/>

### 레디스 설정 이용한 방식

#### AOF 파일 사용여부

```
appendonly yes
```
* **yes** : 레디스 서버 시작 시 이 값이 yes 일때만 AOF 파일을 읽어들입니다.
* no :  AOF 파일이 있어도 읽어들이지 않습니다.

<br/>


#### AOF 파일명 지정

```
appendfilename "appendonly.aof"
```
디렉토리는 `dir`로 지정된 워킹디렉토리를 따름

<br/>

#### AOF에 기록되는 시점 지정

```
appendfsync everysec
```
* always : 명령 실행 시 마다 AOF에 기록. 데이터 유실은 거의 없지만 성능이 매우 떨어짐.
* **everysec** : 1초마다 AOF에 기록. 권장
* no : AOF에 기록하는 시점을 OS가 정함(일반적으로 리눅스의 디스크 기록 간격은 30초). 데이터가 유실될 수 있음..

<br/>

#### AOF rewrite 설정

```
auto-aof-rewrite-percentage 100
```
* AOF 파일 사이즈가 특정 퍼센트 이상 커지면 rewrite 한다.
* 비교 기준은 레디스 서버가 시작할 시점의 AOF파일 사이즈이다.
* 0으로 설정하면 rewrite를 하지 않는다


``` 
auto-aof-rewrite-min-size 64mb
```
* AOF 파일 사이즈가 64mb 이하면 rewrite를 하지 않습니다.  
* 파일이 작을때 rewrite가 자주 발생하는 것을 막아줍니다.

<br/>

### REDIS-CLI을 이용한 방식
#### rewrite 하기

AOF Rewrite는 BGREWRITEAOF 명령으로 명시적으로도 수행시킬 수 있다

```
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
```

<br/>

***

<br/>

## RDB
* RDB는 특정 시점의 메모리에 있는 데이터 전체를 바이너리 파일로 저장하는 것이다.
* AOF 파일보다 사이즈가 작다. 따라서 로딩 속도가 AOF 보다 빠르다. 

![RDB](https://raw.githubusercontent.com/rmcodestar/rmcodestar.github.io/master/public/img/20190427-rdb.jpeg)

<br/>


### 레디스 설정 이용한 방식
#### 저장 시점 정하기

```
save 900 1     #900초(15분) 동안 1번 이상 key 변경이 발생하면 저장 
save 300 10    #300초(5분) 동안 10번 이상 key 변경이 발생하면 저장 
save 60 10000  #60초(1분) 동안 10000번 이상 key 변경이 발생하면 저장 
```
* SAVE 조건은 여러 개를 지정할 수 있고, 모두 or 조건이다. 즉 어느 것 하나라도 만족하면 저장한다.
* RDB를 저장하지 않으려면 redis.conf 에서 SAVE를 모두 주석 처리하면 된다

<br/>


#### RDB 파일명 지정

```
dbfilename dump.rdb
```

디렉토리는 `dir`로 지정된 워킹디렉토리를 따름

<br/>


#### RDB 저장 실패시 데이터 읽기 여부
RDB 파일 저장이 실패했을 경우 데이터를 받아 들일지 말지를 정하는 파라미터이다

```
stop-writes-on-bgsave-error yes
```
* 이 값이 yes 일때, 레디스는 RDB 파일을 디스크에 저장하다 실패하면, 모든 쓰기 요청을 거부한다. 쓰기에 문제가 발생했으니, 빨리 조치를 취하라는 의미다. default는 yes이다.
* 이 값을 no 로 설정하면, 디스크 저장에 실패하더라도, 레디스는 쓰기 요청을 포함한 모든 동작을 정상적으로 처리한다.
* 디스크 쓰기에 실패하는 경우는 여유 공간이 부족하거나, 권한(permission) 부족, 디스크 물리적 오류 등이 있을 수 있다.
* 이 파라미터는 save 이벤트에만 해당한다. BGSAVE 명령을 직접 입력했을 경우에는 해당하지 않는다.

<br/>

### REDIS-CLI을 이용한 방식

#### RDB 파일 생성

명령으로도 RDB 파일을 생성할 수 있다. BGSAVE 또는 SAVE 이다.

1. SAVE
2. BSAVE

```
127.0.0.1:6379> BGSAVE
Background saving started
```

파일 쓰기 작업은 Child process가 생성되어 background로 실행되므로 쓰기 작업 중에도 레디스 서버는 클라이언트의 명령을 정상적으로 처리할 수 있다

<br/>

## DB 복구
레디스 서버가 시작될 때 데이터 파일을 읽는다.
어떤 데이터 파일을 읽을지는 레디스 설정파일의 `appendonlyfile`설정을 따른다

*  `appendonlyfile yes` : aof 파일을 읽음
*  `appendonlyfile no` :  rdb 파일을 읽음


<br/>



## 어느 것을 선택할까

==>  AOF(Append Only File)

> AOF를 기본으로 하고, RDB를 Option으로 한다. AOF 시간 설정은 everysec로 하고, AOF Rewrite를 사용한다. AOF를 사용해도 성능에 거의 영향을 미치지 않습니다. 



<br/>


## reference
http://redisgate.kr/redis/configuration/persistence.php
