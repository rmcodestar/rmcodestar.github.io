---
layout: post
title: DockerFile
category: docker
tag : [docker]
---

## DockerFile
* 컨테이너에서 수행해야할 작업을 명시
* Doc : https://docs.docker.com/engine/reference/builder/

## Format
```
# Comment
INSTRUCTION arguments
```
* 명령(INSTRUCTION)은 대소 문자를 구분하지 않습니다
* 순서대로 명령이 실행됨
* Dockerfile 는`FROM` 명령으로 시작해야함
* 기본 escape 문자는 '₩'

## DockeFile 명령어

### FROM
* 생성할 이미지의 베이스가 될 이미지
* Dockerfile은 `FROM`명령어로 시작되어야 함

### MAINTAINER
* 이미지를 생성한 개발자의 정보
* docker 1.13.0 버전 이후로 Deprecated

### LABEL
```
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```
* 이미지에 메타데이터를 추가함
* 키:값 형태로 저장되며 여러 개 저장 가능
* 추가된 메타데이터는 `docker inspect`로 확인 가능

### RUN
* 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행

### WORKDIR
* 명령어를 실행할 디렉터리
* 여러 번 실행 가능

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd   # /a/b/c에서 pwd가 실행됨
```

### ADD
```
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```
* 파일을 이미지에 추가함

### EXPOSE
* Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정


### CMD
* 컨테이너가 시작될 때마다 실행할 명령어(커맨드)를 설정
* DcokerFile에서 한 번만 실행됨, 둘 이상 있을 경우 마지막 항목만 적용됨



## Dockerfile 빌드
```shell
# docker build -t mybuild:0.0 ./
```
* `-t` : 생성될 이미지의 이름을 설정하는 옵션
* 인자에는 dockerfile이 저장된 경로를 입력함
