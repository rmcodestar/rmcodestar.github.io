---
layout: post
title:  confluent schema registry의 schema 호환성
category: Kafka
tag: [Kafka, confluent schema registry]
---


**궁금증**

Schema version이 달라지면 어떻게 호환성이 관리될까? 

시간이 지나면 초기 개발과 달리 스키마에 필드를 추가하거나 삭제를 해야할 일이 발생할 수 있다.

<br> 

그렇다면 버전이 업이 되고 producer와 consumer에서는 어떤 버전을 처리하고 어떤 순서로 먼저 반영을 하는 것이 좋을까?

<br>

**호환성 유형**

Confluent Schema Registry 기본 호환성 유형은 `BACKWARD`

| Compatibility Type    | Changes allowed                                   | Check against which schemas  | Upgrade first |
| :-------------------- | :------------------------------------------------ | :--------------------------- | :------------ |
| `BACKWARD`            | fields 삭제 가능 <br> optional fields 추가 가능       | 마지막 버전                     | `Consumers`   |
| `BACKWARD_TRANSITIVE` | fields 삭제 가능 <br> optional fields 추가 가능       | 모든 이전 버전                   | `Consumers`   |
| `FORWARD`             | fields 추가 가능 <br> optional fields 삭제 가능       | 마지막 버전                     | `Producers`   |
| `FORWARD_TRANSITIVE`  | fields 추가 가능 <br> optional fields 삭제 가능       | 모든 이전 버전                   | `Producers`   |
| `FULL`                | optional fields 추가 가능 <br> optional fields 삭제 가능| 마지막 버전                   | 순서 상관 없음    |
| `FULL_TRANSITIVE`     | optional fields 추가 가능 <br> optional fields 삭제 가능| 모든 이전 버전                | 순서 상관 없음    |
| `NONE`                | 모든 변경 사항이 가능함                                | 호환성 검사 비활성화              | Depends       |


해당 스키마의 호환성 유형이 `BACKWARD`이라면 새 스키마를 사용하는 `Consumer`가 마지막 스키마로 생성 된 데이터를 읽을 수 있다.

먼저 모든 `Consumer`를 업데이트된 버전을 consume할 수 있게 처리를 한 후 그 다음에 업데이트된 버전으로 produce할 수 있도록 `producer`를 처리하는 순서로 진행한다.

하지만 이경우엔 필드 삭제나, optional 필드를 추가하는 경우만 호환성을 유지할 수 있다.

<br>


**Reference**
* [schema-evolution-and-compatibility](https://docs.confluent.io/current/schema-registry/avro.html#schema-evolution-and-compatibility)
