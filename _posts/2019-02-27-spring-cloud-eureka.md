---
layout: post
title: 스프링 클라우드 유레카
category: Spring
tag: [Spring, Spring cloud, Eureka]
---


## Eureka 서버/클라이언트 실습


### 유레카 (spring-cloud-starter-eureka) - 서비스 디스커버리
* 클라이언트는 항상 애플리케이션의 일부로 원격의 디스커버리 서버에 연결하는 일을 한다.
* 연결되면, 서비스 이름과 네트워크 위치를 담은 등록 메시지를 보낸다.


spec
* spring-boot : 2.1.2.RELEASE
* spring-cloud-dependencies : Finchley.SR2

pom.xml
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

<BR>


## 서버와 클라이언트
서버
* 전용 스프링 부트 애플리케이션으로 실행한다
* 서버 API는 등록된 서비스의 목록을 수집하기 위한 API와 새로운 서비스를 네트워크 위치 주소와 함께 등록하기 위한 API로 구성된다

클라이언트
* 마이크로서비스 애플리케이션에 의존성을 포함해 사용한다.
* 클라이언트는 애플리케이션 시작 후 등록과 종료 전 등록 해제를 담당하고 유레카 서버로부터 주기적으로 최신 서비스 목록을 가져온다.


<BR>


## 서버 측에서 유레카 서버 실행하기
* spring-cloud-stgarter-netflex-eureka-server 의존성 추가

유레카 서버 활성화

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class StudySpringCloudApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudySpringCloudApplication.class, args);
    }

}
```

<BR>

컨피규레이션 설정(application.properties)
```properties
server.port=${PORT:8761}
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

* http://localhost:8761/

<BR>

## 클라이언트로 유레카 시작하기
pom.xml
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

<BR>

* 서버측에서 서비스를 확인해보자

http://localhost:8761/eureka/apps

```xml
<applications>
    <versions__delta>1</versions__delta>
    <apps__hashcode/>
</applications>
```

<BR>

## 유레카 클라이언트  활성화하기
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class StudySpringCloudApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudySpringCloudApplication.class, args);
    }

}
```

<BR>


컨피규레이션 설정하기
```properties
spring.application.name=client-service
server.port=${PORT:8081}
eureka.client.serviceUrl.defaultZone=${PEUREKA_URL:http://localhost:8761/eureka/}
```
* `EUREKA_URL` : 디스커버리 서버 네트워크 주소
*  클라이언트 리스닝 포트는 PORT 환경변수로 정의할 수 있다


<BR>


이제 두 개 인스턴스를 실행해보자
```
java -jar -DPORT=8081 study-spring-cloud-eureka-client-0.0.1-SNAPSHOT.jar
java -jar -DPORT=8082 study-spring-cloud-eureka-client-0.0.1-SNAPSHOT.jar
```

<BR>

아래 서버측 API를 통해 두 인스턴스가 등록되었는지 확인해보자
http://localhost:8761/eureka/apps

```xml
<applications>
    <versions__delta>1</versions__delta>
    <apps__hashcode>UP_2_</apps__hashcode>
    <application>
        <name>CLIENT-SERVICE</name>
        <instance>...</instance>
        <instance>...</instance>
    </application>
</applications>
```

<BR>

## 인스턴스 설정

인스턴스 아이디 : `${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}`

> ex : 192.168.0.106:client-service:8082

다르게 설정하고 싶다면
```properties
eureka.instance.instanceId=${sprnig.application.name}-${SEQUENCE_NO}
```

> client-server-1
