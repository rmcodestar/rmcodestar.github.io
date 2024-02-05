---
layout: post
title: OutputCapture
category: Spring
tag: [Spring, Junit]
---

### OutputCapture (2.1.X 버전 이하에서만 지원)
* spring boot test에서 지원
* **2.2.0 이후 derprecated됨**
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

로깅도 테스트할 수 있게되었다!


### CapturedOutput (2.2.X부터 지원)
* spring boot 2.2.X부터 해당 방식을 사용해야함
* https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/system/CapturedOutput.html
```java
@ExtendWith(OutputCaptureExtension.class)
```
`OutputCaptureExtension`을 사용하고 테스트 메서드에 `CapturedOutput`인자로 받아서 테스트
```java
@Test
public void testcase(CapturedOutput capturedOutput) {
	assertThat(capturedOutput).contains("hello, world");
}
```


