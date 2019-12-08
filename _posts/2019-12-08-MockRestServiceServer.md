---
layout: post
title: MockRestServiceServer를 이용한 restTemplate Test
category: spring
tag: spring, test
---

### MockRestServiceServer

`restTemplate`를 이용하여 api 요청을 하는 메서드를 테스트 하고 싶다면  `MockRestServiceServer`를 활용해보자

Mock server 인스턴스를 생성해서 원하는 응답값을 제어할 수 있다.

(`spring boot` 1.4+부터 지원한다고 한다.)

<br>

예를 들어 id를 이용하여 유저정보를 조회해온다고 할 때

<br>

```java
package com.study.springboot.springbootstarter.service;

import com.study.springboot.springbootstarter.model.User;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class UserService {
    @Value("${user-api.get-user-api-url}")
    public String url;

    private RestTemplate restTemplate;

    public UserService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    public User getUser(int id) {
        return restTemplate.getForObject(url, User.class, id);
    }
}
```

<br>

`getUser`메서드에 대한 테스트케이스를 작성해보자.

<br>

```java
package com.study.springboot.springbootstarter.service;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.study.springboot.springbootstarter.model.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.client.RestClientTest;
import org.springframework.context.annotation.Description;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.client.MockRestServiceServer;
import org.springframework.test.web.client.match.MockRestRequestMatchers;
import org.springframework.test.web.client.response.MockRestResponseCreators;
import org.springframework.web.client.HttpClientErrorException;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@RestClientTest(value = UserService.class)
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Autowired
    private MockRestServiceServer mockRestServiceServer;

    @Autowired
    private ObjectMapper objectMapper;

    @Description("성공적으로 유저 정보를 가져온 경우")
    @Test
    public void testcase1() throws JsonProcessingException {
        //Given
        User expected = new User(1, "test");
        String responseJson = objectMapper.writeValueAsString(expected);

        mockRestServiceServer.expect(MockRestRequestMatchers.anything())
                .andRespond(MockRestResponseCreators.withSuccess(responseJson, MediaType.APPLICATION_JSON));

        //When
        User actual = userService.getUser(id);

        //Then
        assertNotNull(actual);
        assertEquals(actual.getId(), expected.getId());
        assertEquals(actual.getName(), expected.getName());
    }
}
```

<br>

우선 `@RestClientTest` 어노테이션과 value에는 테스트할 sut를 입력한다.

그리고 테스트시 mock된 mockRestServiceServer bean이 응답해야할 값을 `andResponse`메서드 안에 정의해준다.

<br>

그러면 편하게 테스트할 수 있다!

<br>

> @RestClientTest 어노테이션이 있을 경우, 
>
> 
> 테스트를 위한 전체 자동 구성을 비활성화하고 rest 클라이언트 테스트에 관련된 구성만 한다고 한다.

<br>

### reference 

* https://www.baeldung.com/restclienttest-in-spring-boot
