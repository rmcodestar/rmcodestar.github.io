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
