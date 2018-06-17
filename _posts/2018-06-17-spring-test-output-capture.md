---
layout: post
title: OutputCapture
category: spring boot test
tag: [spring boot test, junit]
---

### OutputCapture
* spring boot test에서 지원
* junit 작성 시 `System.out`와 `System.err`을 가지고 테스트를 할 수 있다
* OutputCapture를 사용하기 위해서 `@Rule` 어노테이션을 사용해야 한다.
* 비교할 출력을 OutputCapture의 인스턴스의  `toString`을 해서 가져올 수 있다.

* [example](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html)

```java
import org.junit.Rule;
import org.junit.Test;
import org.springframework.boot.test.OutputCapture;

import static org.hamcrest.Matchers.*;
import static org.junit.Assert.*;

public class MyTest {

	@Rule
	public OutputCapture capture = new OutputCapture();

	@Test
	public void testName() throws Exception {
		System.out.println("Hello World!");
		assertThat(capture.toString(), containsString("World"));
	}

}
```

> 로깅도 테스트할 수 있게 되었으나 과연 단위테스트에 로깅까지 할 필요가 있을지는 의문
>
> 알고리즘 문제 테스트시엔 좋은 것 같음 (보통 표준출력으로 정답제출하므로)
