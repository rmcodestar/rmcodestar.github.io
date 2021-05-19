---
layout: post
title:  kafka local에서 테스트 해보자
category: kafka
tag: [kafka, docker]
---

## kafka 로컬 테스트

kafka document : https://kafka.apache.org/documentation/



### local에 kafka 설치하기

* https://github.com/wurstmeister/kafka-docker
*  [docker-compose로-kafka를-로컬에-띄워보자](http://www.kwangsiklee.com/2017/03/docker-compose로-kafka를-로컬에-띄워보자/)



### kafka docker container 실행 및 중단

컨테이너 프로세스 확인

```shell
$ docker ps -a
```



zookeeper, kafka 실행

- IMAGE `wurstmeister/zookeeper` `kafka-docker_kafka` 인 컨테이너 실행
- zookeeper는 2888 포트에, kafka는 9092 포트에 바인딩되어있음

```shell
$ docker start {container-id}
```



zookeeper, kafka 중단

```shell
$ docker stop {container-id}
```



kafka container 접속

```shell
$ docker exec -it {kafka-docker_kafka container-id} /bin/bash
```

* /opt/kafka/bin : kafka 관련 shell 있음
* /kafka/kafka-log-{container-id} : offset, topic 로그가 있음
  * ex. `__consumer_offset_0`


<br>


## QuickStart

#### 1. topic 생성하기

test-topic을 만들어보자 (파티션은 1개고 복제인수도 1개인)

```shell
$ ./kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test-topic
```



#### 2. console producer 사용해서 메시지 보내기

```shell
$./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic
This is a message
This is another message
```



#### 3. console consumer 사용하기

```shell
$ ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

아까 보낸 메시지들이 아래와 같이 뜰 것 이다

```
This is a message
This is another message
```


<br>


## kafka command-line tool

usecase별로 시나리오 작성



#### consumer의 정보 확인하기

* 파티션, offset, lag 정보

```shell
$./kafka-consumer-groups.sh --bootstraps-server localhost:9092 --group ${consumer-group} --describe
```



ex. `test-consumer` 컨슈머그룹이 어떤 토픽을 하고 있나 확인!

* `test-topic` 토픽을 구독하고 있고 해당 토픽은 파티션이 3개임을 알 수 있다.
* client-id를 통해 해당 그룹에 client가 2개 떠있음을 알 수 있다.

```shell
$./kafka-consumer-groups.sh --bootstraps-server localhost:9092 --group test-consumer --describe

GROUP		TOPIC	PARTITION	CURRENT-OFFSET	LOG-END-OFFSET	LAG	CONSUMER-ID	HOST	CLIENT-ID
test-consumer	test-topic	2	0	0	0	consumer-test-consumer-1-{UUID}	{ip}	consumer-test-consumer-1
test-consumer	test-topic	0	0	0	0	consumer-test-consumer-1-{UUID}	{ip}	consumer-test-consumer-1
test-consumer	test-topic	1	0	0	0	consumer-test-consumer-2-{UUID}	{ip}	consumer-test-consumer-2
```





#### topic 정보 확인하기

* 해당 토픽의 파티션 개수, replicationFactor, ISR 정보를 알 수 있다

```shell
$./docker-topics.sh --bootstrap-server localhost:9092 --topic ${topic} --descibe
```



ex. test-topic의 정보를 확인.

* 해당 토픽은 파티션이 4개 있다
* 복제인수는 2개이므로 2개의 브로커에 저장된다. leader, isr도 알 수 있다.

```shell
$./docker-topics.sh --bootstrap-server localhost:9092 --topic test-topic --descibe

Topic:test-topic	PartitionCount:4	ReplicationFactor:2	Configs:min.insync.replica=2
topic:test-topic	Partition: 0 Leader: 3 Replicas: 3,4 Isr : 3,4
topic:test-topic	Partition: 1 Leader: 4 Replicas: 4,5 Isr : 5,4
topic:test-topic	Partition: 2 Leader: 5 Replicas: 5,1001 Isr : 5,1001
topic:test-topic	Partition: 3 Leader: 1001 Replicas: 1002,2 Isr : 1001,2
```

#### 해당 토픽 파티션 개수 변경하기
ex. test 토픽의 partiions 3으로 변경
```shell
$./kafka-topics.sh --bootstrap-server localhost:9092 --topic test --alter --partitions 3
```


#### 특정 consumer group Offset 조정하기
단, 조정할 때 해당 consumer application 실행 중이면 안되므로 종료해야한다.
```shell
$./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group ${consumer-group} --topic ${topic} --reset-offsets {--to-earliest|--to-latest|--to-offset } --execute
```

* offset reset 방법 (그 외도 있으나 편할 것 같은 옵션 위주로)
  - `--to-datetime <String: datetime>` : Reset offsets to offsets from datetime. 
    - Format: `YYYY-MM-DDTHH:mm:SS.sss`
  - `--to-earliest` : Reset offsets to earliest offset.
  - `--to-latest` : Reset offsets to latest offset.
  - `--to-current` : Resets offsets to current offset.
  - `--to-offset <Long: offset>` : Reset offsets to a specific offset.



#### 특정 옵셋으로 조정하기

ex.`test-consumer`그룹이 consume하고 있는 `test-topic` offset을 100으로 조정하기

```shell
$./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-consumer --topic test-topic --reset-offsets --to-offset 1000 --execute

TOPIC	PARTITION	NEW-OFFSET
test-topic	0 1000
```

아래와 같은 에러가 뜬다면 consumer 애플리케이션을 종료하고 시도하자.
```
 Error: Assignments can only be reset if the group 'test-consumer' is inactive, but the current state is Stable.
```

#### avro console consumer 사용하기

https://docs.confluent.io/1.0.1/quickstart.html


ex.`testTopic`토픽 avro console consumer로 처음부터 구독하기
```shell
$./kafka-avro-console-consumer --topic {topicName} --bootstrap-server localhost:9092 --property schema.registry.url="{schemaRegistryUrl}" --from-beginning
```




