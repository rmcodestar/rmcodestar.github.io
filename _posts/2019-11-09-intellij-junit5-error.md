---
layout: post
title: intellij junit5, gradle 환경에서 no tests found 에러
category: [ETC, 개발환경]
tag: [Intellij]
---


> intellij 2019.2.4
>
> junit 5

### junit5 사용시 테스트 돌릴때 아래와 같은 에러가 발생..

#### 에러메시지
```
no tests found for given includes [xxxx.xxxTest](filter.includetestsmatching)
```

<br>

위 에러가 발생하는 케이스는 junit5 관련 어노테이션이 있는 TC에서 발생..
인텔리제이에서 junit5를 인식못하는 것이라 판단

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(MockitoExtension.class)
```

<br>

#### 해결방법 

인텔리제이 설정 변경
`Preferences` > `Build, Execution, Deployment > Build Tools > Gradle` > 오른쪽 영역 `Build and run using`, `Run test using` 옵션을 `gradle`이 아닌 `Intellij IEDA`로 변경 후 정상 동작함

<br>

* `Run test using` : `Intellij IEDA`

***

* 관련글 : [Intelij 2019.1 update brakes JUnit tests](https://stackoverflow.com/questions/55405441/intelij-2019-1-update-brakes-junit-tests)
