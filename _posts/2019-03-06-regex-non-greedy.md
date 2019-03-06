---
layout: post
title: 정규표현식의 non-greedy 매칭
category: 정규표현식
tag: [정규표현식, non-greedy]
---


greedy와 non-greedy 패턴 매칭의 차이는 무엇일까?


<br>


`1234`라는 문자열을 `\d+(숫자 한글자 이상)`패턴 매칭을 한다고 하자.

최대한 많이 매칭되는 결과는 `1234`이다. --> greedy matching

가장 최소한으로 매칭되는 결과는 `1`일 것이다. --> non-greedy matching


<br>


이 둘의 차이를 몰랐던 나는 아래의 케이스에서 삽질을 하고 있었다.


<br>

## 삽질한 예시

아래 html에서 a링크의 href를 그룹매칭해서 가져오고 싶다고 해보자

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

> 수량자(*, +) 바로 뒤에 사용하면, 기본적으로 탐욕스럽던(가능한 한 많이 대응시킴) 수량자를 탐욕스럽지 않게(가능한 가장 적은 문자들에 대응시킴) 한다.


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
