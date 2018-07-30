---
layout: post
title: Proxy server란
category: 네트워크
tag: [네트워크, proxy]
---

## 프록시 서버란?
> 서버와 클라이언트 사이에서 중계기로서 대리로 통신을 수행하는 기능을 가리켜 `프락시(proxy)`, 그 중계 기능을 하는 것을 `프락시 서버(proxy server)`라고 부른다.

## 프록시 서버의 종류
* Forward proxy server
* Reverse proxy server ✨
* Open proxy server

<br>

***
## Forward proxy server
![forward proxy server](https://upload.wikimedia.org/wikipedia/commons/thumb/1/19/Forward_proxy_h2g2bob.svg/320px-Forward_proxy_h2g2bob.svg.png)

* client가 destination server에 전송하기 위해선 proxy 서버를 통해야 함
* proxy 서버는 client가 요청을 해야만 destination server를 알 수 있다.


## Reverse proxy server ✨
![reverse proxy server](https://upload.wikimedia.org/wikipedia/commons/thumb/6/67/Reverse_proxy_h2g2bob.svg/280px-Reverse_proxy_h2g2bob.svg.png)

* 클라이언트가 프록시 서버에게 요청하면 프록시서버가 뒷단의 전송서버의 응답값을 가져와 대신 클라이언트에게 전달함
* 클라이언트는 프록시 뒤의 destination 서버를 알지 못한다


## Open proxy server
![open proxy server](https://upload.wikimedia.org/wikipedia/commons/thumb/2/27/Open_proxy_h2g2bob.svg/280px-Open_proxy_h2g2bob.svg.png)

* 모든 인터넷 사용자가 액세스 할 수있는 전달 프록시 서버
* 익명 공개 프록시는 사용자가 자신의 IP 은폐 할 수 있도록 하는데 사용됨

<br>

***

## Reference
* [wikipedia - proxy server](https://en.wikipedia.org/wiki/Proxy_server)
