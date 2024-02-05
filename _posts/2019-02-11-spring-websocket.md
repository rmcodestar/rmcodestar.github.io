---
layout: post
title: Spring websocket으로 간단 채팅 프로그램 만들기
category: Spring
tag: [Spring, Websocket, STOMP, sockjs]
---

# Spring websocket으로 채팅만들기



>spring 4버전부터 웹소켓을 지원하게 되었다.
>
>아래 스펙으로 간단한 채팅 프로그램을 만들어보자
>
>=> 스프링 + websocket + STOMP

<br>

maven dependency

* spring boot
* spring-boot-starter-websocket

javascript library

* [sockjs-client](https://github.com/sockjs/sockjs-client)
* [stomp-websocket](https://github.com/jmesnil/stomp-websocket)

<br>

> SockJs란?
>
> SockJS는 WebSocket과 유사한 객체를 제공하는 브라우저 JavaScript 라이브러리입니다. SockJS는 브라우저와 웹 서버간에 낮은 대기 시간, 전이중, 도메인 간 통신 채널을 생성하는 일관된 크로스 브라우저 Javascript API를 제공합니다.
>
> 후드 아래 SockJS는 먼저 원시 웹 소켓을 사용하려고 시도합니다. 실패 할 경우 다양한 브라우저 별 전송 프로토콜을 사용하여 WebSocket과 같은 추상화를 통해 제공 할 수 있습니다.


<br>

***


## 1. spring에서 web socket 활성화시키기

`@EnableWebSocketMessageBroker` 으로 Websocket 기능을 활성화시킨다.

그리고 아래와 같은  `AbstractWebsocketMessageBrokerConfigurer`을 상속한 Configuration이 필요하다.

<br>

Websocket api를 바로 이용하지 않고 STOMP를 통해서 설정을 할 것이다.

<br>

1. `configureMessageBroker` : 메시지 브로커에 관련된 설정을 한다
2. `registerStompEndpoints` : `SockJs Fallback`을 이용해 노출할 STOMP endpoint를 설정한다.


<br>


```java
@Configuration
@EnableWebSocketMessageBroker
public class ChatConfiguration extends AbstractWebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/chat", "/participants");
        registry.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry stompEndpointRegistry) {
        stompEndpointRegistry.addEndpoint("/endpoint").withSockJS();
    }
}
```

<br>

각 설정을 자세히 설명하자면

```java
stompEndpointRegistry.addEndpoint("/endpoint").withSockJS();
```

sockJs 클라이언트가 Websocket 핸드셰이크를 하기 위해 연결할 endpoint를 지정할 수 있다.

클라이언트가 연결되고 `http://localhost:8080/endpoint/info?t=12312312`으로 웹소켓 통신이 가능한지 확인한 후,

응답이 websocket:true 이면 웹소켓 연결된다.

<br>

```java
registry.setApplicationDestinationPrefixes("/app");
registry.enableSimpleBroker("/chat", "/participants");
```

`ApplicationDestinationPrefixes`를 지정하면 대상 헤더가 시작되는 STOMP 메시지는 해당 클래스의 메서드로 라우팅된다.

<br>

예를 들어 클라이언트가 SEND 프레임을 다음과 같이 보낼 때 

```
>>> SEND
destination:/app/message
content-length:83

{"roomId":"1","userId":"6666","message":"123123","date":"2019-02-11T15:04:59.958Z"}
```

<br>

 `@Controller`에서는 `/app` desination prefix를 제외한 경로 `/message`를 `@MessageMapping`하면 된다.

```java
@Controller
public class ChattingController {

    @MessageMapping("/message")
    public void receiveMessage(@Payload ChattingMessage message) {
        //TODO
    }
}
```

<br>

## 2. @Controller에서 메시지 핸들링하기

Controller에서 아래의 두 어노테이션을 이용해서 메시지를 핸들링할 수 있다. 

1. @MessageMapping
2. @SubscribeMapping

<br>

> 기본적으로 @SubscribeMapping 메서드의 반환값은 연결된 클라이언트에게 직접 메시지로 보내지며 브로커를 통과하지 않는다.

<br>

사용예제 : 클라이언트가 `/app/message`로 메시지를 보냈을 경우 

```java
@Controller
public class ChattingController {

    @MessageMapping("/message")
    public void receiveMessage(@Payload ChattingMessage message) {
        //TODO
    }
}
```

<br>

## 3. 메시지 보내기

연결된 클라이언트에게 메시지를 보내고 싶을 땐 어떻게 해야할까?

브로커를 설정하지 않은 경우  `SimpMessagingTemplate`을 주입받아서 사용하면 된다.

<br>

간단하게 컨트롤러에서 해당 HTTP 요청을 받을 때 `SimpMessagingTemplate`의 `convertAndSend`메서드를 호출하게 하면 테스트가 쉽다.

```java
@Autowried
private SimpMessagingTemplate template;

@RequestMapping("/greetings")
public void greet(String greetingMessage) {
    template.convertAndSend("/chat/message", greetingMessage);
}
```

`/chat/message`를 구독하고 있는 클라이언트들에게 greetingMessage이 보내지게 될 것이다.

<br>

>Simple Broker
>
>브로커 설정을 따로 하지 않을 경우 메모리에 클라이언트 구독 정보를 저장하는 Simple broker가 사용되게 된다.
>
>
>
>RabbitMQ나 ActiveMQ등 외부 브로커를 사용하고 싶다면  [documenet](https://docs.spring.io/spring/docs/5.0.3.BUILD-SNAPSHOT/spring-framework-reference/web.html#websocket-stomp)의 4.4.8 external Broker설정을 참고하자

<br>

## 4. sockJs + stomp

javascript sockejs, stomp-websocket을 이용해서 웹소켓 클라이언트를 만들어보자

<br>

먼저 웹소켓 연결을 하기위해 client객체를 생성하고 `connect`를 호출해보자

```javascript
var endPoint = "/endpoint";
var stompClient = Stomp.over(new SockJS(endPoint));
var header = { userId : "testId"};

stompClient.connect(header, function (frame) {
    console.log("connected: " + frame);
});
```

console에는 아래와 같이 로그가 남는다

```
connected: CONNECTED
heart-beat:0,0
version:1.1
```

<br>

그리고  `/chat/messages.1` 채널을 구독해보자.

두 번째 인자인 callback에서는 구독한 채널에서 메시지를 수신받았을 때 처리할 로직을 작성한다.

```javascript
stompClient.subscribe("/chat/messages.1", function (response) {
    var data = JSON.parse(response.body);
    console.log(data);
});
```

<br>

그리고 메시지를 보내고 싶을 때는 `send`를 사용한다.

```javascript
var chat = {
    roomId: 1
    , userId: "user01"
    , message: "test messaage"
    , date: new Date()
};

stompClient.send("/app/message", {}, JSON.stringify(chat));
```

<br>

>  자세한 stomp client interface는 아래 document를 참고하자
>
> http://jmesnil.net/stomp-websocket/doc/


<br>


## Reference

* 채팅예제 : https://github.com/rmcodestar/spring-websocket-study
* [spring websocket doc](https://docs.spring.io/spring/docs/5.0.3.BUILD-SNAPSHOT/spring-framework-reference/web.html#websocket)

