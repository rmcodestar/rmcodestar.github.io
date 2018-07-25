---
layout: post
title: docker 명령어
category: docker
tag : [docker]
---
## 도커 엔진 버전 확인
```shell
$ which docker
```
결과
```
/usr/local/bin/docker
```

## 도커 엔진 버전 확인
```shell
$ docker version
```
결과
```
Docker version 18.03.1-ce, build 9ee9f40
```

## 도커 이미지
### 이미지 목록 출력
```shell
$ docker images
```

```
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
mysql                     5.7                 0d16d0a97dd1        2 months ago        372MB
ubuntu                    14.04               8cef1fa16c77        2 months ago        223MB
centos                    7                   e934aafc2206        3 months ago        199MB
```

### 이미지 pull
```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

## 도커 컨테이너
### 컨테이너 목록 출력
```
docker ps [OPTIONS]
```
* 정지된 컨테이너만 출력
* 옵션 `-a`을 주면 모든 컨테이너를 출력

### 컨테이너 실행
* 정지된 컨테이너만 출력
```
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...] 
```
* 대화식 프로세스 (예 : 쉘)의 경우, -i -t컨테이너 프로세스에 tty를 할당하기 위해 함께 사용해야합니다 . 
* `--name` 옵션을 이용해 컨테이너명으로 실행할 컨테이너 찾음.
