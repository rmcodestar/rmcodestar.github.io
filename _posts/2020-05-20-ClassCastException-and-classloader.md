---
layout: post
title:  ClassCastException 삽질
category: 삽질
tag: [삽질]
---

### ClassCastException 삽질..

<br>

#### **문제상황**

confluent schema registry에 있는 스키마로 TestDomain을 java pojo로 역직렬화하여 카프카이벤트를 consume하는 개발을 하고있었다.

```java
@KafkaListener(topics = "test-topic")
public void listen(ConsumerRecord<String, TestDomain> record) {
        testService.doAnything(record.value());
}
```

그런데 `ClassCastExeption`이 발생. (클래스명도 path도 동일한데 왜 캐스팅을 못하니 😭)

```
Caused by: java.lang.ClassCastException: com.study.domain.TestDomain cannot be cast to com.study.domain.TestDomain
```

<br>

왜일까?🤔 (= 내가 뭘 잘못했을까?)

<br>

**나의 추측**

1. `TestDomain` java를 못찾았다. -> ❌ 그렇다면 에러 메시지가 달랐을 것
2. `serialVersionUID`가 다른 객체 -> ❌ 
3. `SpecipicRecord`가 아니라 `GenericRecord`라 casting이 안되었다. -> ❌ 디버깅시 해당 impl은 SpecipicRecord가 맞았다.

<br>


**방황 중에 발견한 강같은 글**

https://stackoverflow.com/questions/46848557/same-class-not-assignable-classloader-for-same-class-different

`classloader`..!


<br>


클래스로더가 다를 수도 있단 생각이 들었다.

그런 무엇때문에 클래스로더가 다르단 말인가?

<br>


**spring boot devtools**

devtools 에서는 `A base classloader`와 `A Retart classloader` 2개가의 클래스로더가 있고 애플리케이션에서 개발된 클래스는 `A restart classloader`를 사용한다.

> Restart vs Reload
>
> The restart technology provided by Spring Boot works by **using two classloaders**. Classes that do not change (for example, those from third-party jars) are loaded into **a *base* classloader**. Classes that you are actively developing are loaded into **a *restart* classloader**. When the application is restarted, the *restart* classloader is thrown away and a new one is created. This approach means that application restarts are typically much faster than “cold starts”, since the *base* classloader is already available and populated.

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-devtools-restart


<br>


**결론**

그래서 avro desrializer에서 사용한 클래스로더와 애플리케이션 내 ListenContainer에서 해당 클래스를 로더한 클래스로더가 달랐던 게 아닐까?

<br>


**처방**

local에서 사용하는 devtools dependecy를 삭제하여 해결!



<br>


**참고**

검색해보니 동일한 경험?을 한 글이 있었다

* [spring boot devtools 클래스로더 이슈](https://brunch.co.kr/@springboot/212)

