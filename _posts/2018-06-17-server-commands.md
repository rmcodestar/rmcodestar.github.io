---
layout: post
title: 리눅스 서버 명령어
category: 리눅스
tag: [리눅스]
---

### disk usage

#### 디스크 사용량 확인
```
df -h
```
* options 
  * `-h` : 사람이 읽을 수있는 형식으로 표시된 디스크 공간 (KB, MB, GB)

***

### 프로세스 확인

#### 아파치/톰캣 프로세스 확인
* 아파치 프로세스 확인
```
ps -ef | grep httpd
```

* tomcat 프로세스 확인
```
ps -ef | grep java
```

#### 아파치 프로세스 수 확인
```
ps aux | grep httpd | wc -l
```
* `ps aux` : 현재 실행 중인 프로세스의 CPU, MEM 사용률, 프로세스 상태 코드 등 확인
* `grep httpd` : 표준 입력에서 특정 문자열인 'httpd'가 포함된 행 출력
* `wc -l` : 행수 세기

***

### 로깅
#### 실시간 로그 모니터링
```
tail -f access.log.2018-06-17
```
파일의 마지막 부분을 출력함
* `-f` : 마지막 10 라인을 실시간을 출력 

#### 특정 컬럼 출력 (`awk`이용)
```
tail -f access.log.2018-06-17 | awk '{print $7, $9, $5}'
```

