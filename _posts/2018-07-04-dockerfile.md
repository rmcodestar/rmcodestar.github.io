---
layout: post
title: DockerFile
category: Docker
tag : [Docker]
---

## DockerFile
* 컨테이너에서 수행해야할 작업을 명시
* Document: https://docs.docker.com/engine/reference/builder/

### Format
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
* `docker run`명령어의 이미지 이름 뒤에 입력하는 커맨드와 같은 역할
  * Dockerfile의 `CMD`값은 `docker run`의 커맨드가 있을 경우 덮어쓰기 됨
* 일반 형식과 배열 형식의 차이점
```
CMD echo test
# --> /bin/sh -c echo test
```

```
CMD ["echo", "test"]
# --> echo test
```

### ENV
* Dockerfile에서 사용될 환경변수를 지정
* 빌드된 이미지로 컨테이너를 생성할 경우 이 환경변수를 사용할 수 있음
* 사용방법은 `${ENV_NAME}`, `$ENV_NAME`이렇게 사용가능
```
FROM ubuntu:14.04
ENV APACHE_HOME /home/apps/apache/bin
WORKDIR $APACHE_HOME
```

### VOLUME
* 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉토리

### USER
* USER로 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID를 설정하면 그 아래의 명령어는 해당 사용자의 권한으로 실행하게 됨
* 일반적으로 RUN으로 사용자의 그룹과 계정을 생성한 뒤 사용
```
FROM microsoft/windowsservercore
RUN net user /add patrick
USER patrick
```

### ARG
* build 명령어를 실행할 때 추가로 입력을 받아 Dockerfile 내에서 사용될 변수의 값을 설정
* 기본값을 지정할 수 도 있음

```
FROM ubuntu:14.04
ARG my_value
AGR my_value2:default_value

RUN touch $my_arg/mydir
```

* build 명령어를 실행할 때 `--build-arg` 옵션을 사용해 Dockerfile의 ARG를 입력할 수 있다

```
# docker build --build-arg my_value=/home -t mybuild:0.0 ./
```

### ONBUILD
* 빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile로 생성될 때 실행할 명령어를 추가
* 이는 다른 이미지를 빌드하기 위해 기본으로 사용될 이미지를 빌드하는 경우에 유용
* 부모 이미지의 자식 이미지에만 적용, 자식 이미지는 `ONBUILD` 속성을 상속 받지 않음
```
FROM ubuntu:14.04
RUN echo "hello world"
ONBUILD RUN echo "onbuild" > /onbuild_dir
```
> 위 dockerfile로 빌드된 이미지를 가지고 dockerfile을 생성할 때 /onbuild_dir가 생성되어 있다.

### STOPSIGNAL
* 컨테이너가 정지될 때 사용될 시스템 콜의 종류를 지정, 디폴트는 `SIGTERM`
```
FROM ubuntu:14.04
STOPSIGNAL SIGKILL
```
* `docker run` 명령어에서 `--stop-signal` 옵션으로 컨테이너에 개별적으로 설정할 수도 있다

### HEALTHCHECK
* 이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정
* 상태 체크 실패 시 해당 컨테이너는 `unhealthy`로 설정됨
* 상태 체크에 대한 로그는 컨테이너의 정보에 저장됨. `docker inspect`의 `State`항목 중 `Health`의 `Log` 항목을 참고
```
FROM nginx
RUN apt-get update -y && apt-get install curl -y
HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1
```
* `--interval` : 컨테이너의 상태를 체크하는 주기
* `--timeout` : 상태체크에 실패했다고 판단하는 시간
* `--retries` : 상태 체크 실패시 명령어를 반복하는 횟수


### SHELL
* 사용하려는 셸을 따로 지정하고 싶을 때
* 기본적으로 쉘은 리눅스에서는 `/bin/sh -c`, 윈도우에서는 `cmd /S /C`

***

## Dockerfile 빌드
```
docker build [OPTIONS] PATH | URL | -
```
* 옵션

|option|option shorthand|설명|
| --- | --- | --- |
|`--tag`|`-t`|`name:tag` 포맷으로 이미지 이름을 설정|
|`--file`|`-f`|Dockerfile 이름 (디폴트 값 : `PATH/Dockerfile`)|

* ex. 현재 디렉토리에 있는 `Dockerfile`을 이용하여 이미지 이름 `mybuild:0.0`를 빌드하는 예시
```shell
$ docker build -t mybuild:0.0 ./
```

### `.dockerignore`
* git의 `.gitignore`와 같은 기능
* 빌드 시 이 파일에 명시된 이름의 파일을 컨텍스트에서 제외함
* `.dockerigonore`파일은 컨텍스트의 최상위 경로, 즉 build 명령어에서 맨 마지막에 오는 경로인 Dockerfile이 위치한 경로와 같은 곳에 위치해야 함
* example
```
# comment
*/temp*
*/*/temp*
temp?
```


### 이미지 빌드시
* Dockerfile에 기록된 명령어에 해당하는 각 step마다 새로운 컨테이너가 하나씩 생성되며 이를 이미지로 커밋한다.
* 이미지 빌드가 완료되면 Dockerfile의 명령어 줄 수 만큼의 이미지 레이어가 존재하게 된다
* 중간에 임시로 생겻던 컨테이너는 삭제된다

### 이미지 빌드의 Caching
* 같은 이미지 빌드를 할 경우 이전의 이미지 빌드에 사용했던 이미지 레이어를 활용함 --> 캐싱
* 캐싱을 하고 싶지 않을 경우 `--no-cache`옵션을 추가하여 빌드하면 됨
```shell
# docker build --no-cache -t mybuild:0.0 ./
```
