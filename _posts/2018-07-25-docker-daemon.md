---
layout: post
title: docker 데몬
category: docker
tag : [docker]
---

## 도커 데몬이란?
> 도커 엔진은 외부에서 API 입력을 받아 도커 엔진의 기능을 수행하는데, 도커 프로세스가 실행되어 서버로서 입력을 받을 준비가 된 상태를 도커 데몬이라 한다.

* doc - [dockered](https://docs.docker.com/engine/reference/commandline/dockerd/)


<br>


## 도커의 구조
* 서버
  * 실제로 컨테이너를 생성하고 실행하며 이미지를 관리하는 주체, `dockered` 프로세스
* 클라이언트
  * 도커 데몬 API를 이용할 수 있도록   `CLI(command line interface)`를 제공하는 것

## 클라이언트과 도커 데몬이 동작하는 과정
1. 사용자가 docker 명령어를 입력 `#CLI`
2. 클라이언트는 `/var/run/docker.sock` 유닉스 소켓을 사용하여 도커 데몬에게 명령어를 전달
3. 도커 데몬은 이 명령어를 파싱하고 명령어에 해당하는 작업을 수행
4. 수행 결과를 도커 클라이언트에게 반환하고 사용자에게 결과를 출력

## docker version
```
$ docker version
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:13:02 2018
 OS/Arch:      darwin/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.03.1-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   9ee9f40
  Built:        Thu Apr 26 07:22:38 2018
  OS/Arch:      linux/amd64
  Experimental: true
```

## docker daemon monitoring

### events
```
$ docker events
```
* 도커 데몬이 수행한 명령어의 결과를 실시간으로 볼 수 있다
* `filter` 옵션을 사용하여 원하는 정보만 출력하도록 설정할 수 있다. `docker events --filter 'type=image'`
* filter type 종류 : container, image, volume, network, plugin, daemon
* example : `docker run hello-world`를 실행했을 때의 로그
```
2018-10-22T13:27:39.729138259+09:00 image pull hello-world:latest (name=hello-world)
2018-10-22T13:27:39.873871091+09:00 container create 8dcc6169c9d946913d1036cc4582d4e5dca50b51ffc6a58523cb850975c64823 (image=hello-world, name=practical_saha)
2018-10-22T13:27:39.879489387+09:00 container attach 8dcc6169c9d946913d1036cc4582d4e5dca50b51ffc6a58523cb850975c64823 (image=hello-world, name=practical_saha)
2018-10-22T13:27:39.922456696+09:00 network connect b7f57edd9a193009ad3fc1d7ca132c55b1ec6edba5f87763b4c137212af8cbb6 (container=8dcc6169c9d946913d1036cc4582d4e5dca50b51ffc6a58523cb850975c64823, name=bridge, type=bridge)
2018-10-22T13:27:40.379439879+09:00 container start 8dcc6169c9d946913d1036cc4582d4e5dca50b51ffc6a58523cb850975c64823 (image=hello-world, name=practical_saha)
2018-10-22T13:27:40.502177141+09:00 container die 8dcc6169c9d946913d1036cc4582d4e5dca50b51ffc6a58523cb850975c64823 (exitCode=0, image=hello-world, name=practical_saha)
2018-10-22T13:27:40.767295323+09:00 network disconnect b7f57edd9a193009ad3fc1d7ca132c55b1ec6edba5f87763b4c137212af8cbb6 (container=8dcc6169c9d946913d1036cc4582d4e5dca50b51ffc6a58523cb850975c64823, name=bridge, type=bridge)
```

### docker stats 
```
$ docker stats
```
* 실행 중인 모든 컨테이너의 자원 사용량을 스트림으로 출력
* `--no-stream` : 스트림이 아닌 한 번만 출력하는 방식으로 보여주는 옵션
