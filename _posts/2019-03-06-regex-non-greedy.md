---
layout: post
title: 정규표현식의 non-greedy 매칭
category: 정규표현식
tag: [정규표현식, non-greedy]
---

# 정규표현식의 non-greedy 매칭

일치한다고 판단하는 정도의 수준엔 greedy와 non-greedy 가 있다.

정규표현식은 기본적으로 greedy 매칭을 한다고 한다. 

<br>

## 예시

이 둘의 차이를 알아 보기 위해서 간단한 예시로 아래 html에서 a링크의 href를 그룹매칭해서 가져오고 싶다고 해보자

```html
<a href="https://a.com">a</a><br><a href="https://b.com">b</a>
```

<br>


원하는 결과는 아래와 같다

```text
https://a.com
https://b.com
```

<br>


`<a href="(.*)">`와 같이 정규표현식으로 그룹매칭을 한다고 한다면 결과가 어떻게 나올까?


<br>


## Greedy matching

패턴을 `(.*)`으로 하고 아래 TC를 돌려보았다.

```java
    @Test
    public void test() {
        String html = "<a href=\"https://a.com\">a</a><br><a href=\"https://b.com\">b</a>";
        Pattern linkPattern = Pattern.compile("<a href=\"(.*)\">");

        Matcher matcher = linkPattern.matcher(html);

        while(matcher.find()) {
            System.out.println(matcher.group(1));
        }
    }
```

<br>


`<a href="{링크}">`의 {링크}값만 출력될 것이라고 생각했지만 결과는 아래와 같았다

<br>


```text
https://a.com">a</a><br><a href="https://b.com
```

<br>


정규표현식은 기본적으로 greedy 매칭을 하기 때문이다. 

일치하는 것을 최대한 많이 찾으려고 하다보니 위와 같은 결과가 나온 것이다.


<br>


## Non-greedy matching

결과적으로 내가 원했던 것은 non-greedy 었던 것으로 `?`문자를 사용하면 `*`의 탐욕을 멈출 수 있다고 한다.

기존의 패턴에 `?`을 추가하여 `(.*?)`으로 TC를 돌려보자

```java
    @Test
    public void test() {
        String html = "<a href=\"https://a.com\">a</a><br><a href=\"https://b.com\">b</a>";
        Pattern linkPattern = Pattern.compile("<a href=\"(.*?)\">");

        Matcher matcher = linkPattern.matcher(html);

        while(matcher.find()) {
            System.out.println(matcher.group(1));
        }
    }
```

<br>


출력값은 원했던 대로 나왔다.

```text
https://a.com
https://b.com
```

<br>



### 정규표현식 관련 사이트

* 정규표현식 도식화 사이트 : https://regexper.com/
* 정규표현식 테스트 사이트 : https://regex101.com/
