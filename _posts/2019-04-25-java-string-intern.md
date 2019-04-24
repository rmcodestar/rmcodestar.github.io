---
layout: post
title: java string intern
category: java
tag: [java]
---

## String intern이란?

### String pool

일단 `"test"`, `new String("test")`의 차이를 알면 좋다.


<br>


큰 따옴표로 생성한 문자열(리터럴)의 경우엔 string pool에 저장이 되고, `new`로 생성한 경우 heap에 저장이 된다.


<br>


아래 그림을 보면 이해가 쉬울 것이다. 

따옴표로 생성한 s1와 new String해서 만든 s3가 가리키는 곳은 다르다.



![자바 스트링 풀](https://cdn.journaldev.com/wp-content/uploads/2012/11/String-Pool-Java1-450x249.png)



stirng pool에 동일한 문자열을 계속 저장하면 계속 추가될까? 만약 그렇다면 메모리 낭비일 것이다. 

그래서 동일한 문자열을 중복해서 저장되지 않고 저장된 문자열을 호출할 경우 해당 참조값을 반환한다.

따라서 s1와 s2가 같은 곳을 가리키고 있는 것이다.


<br>


### Intern

java의 string `intern`은 string pool에 등록된 문자열의 참조를 반환한다.



아래 예시에서 어떻게 출력될지를 예상해보자

```java
System.out.println("test" == new String("test"));
System.out.println("test" == new String("test").intern());
```

<br>


`new String("test").intern()`은 string pool에 등록된 "test"의 참조를 반환하게 된다.

따라서, 아래와 같이 출력된다.


```
false
true
```

<br>


Reference

* [What is Java String Pool?](https://www.journaldev.com/797/what-is-java-string-pool)
* [string-의-메모리에-대한-고찰](<https://medium.com/@joongwon/string-%EC%9D%98-%EB%A9%94%EB%AA%A8%EB%A6%AC%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0-57af94cbb6bc>)
