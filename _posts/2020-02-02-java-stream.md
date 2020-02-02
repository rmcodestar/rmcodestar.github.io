---
layout: post
title: java stream
category: java
tag: [java, stream]
---

# STEAM



## Steams in java

Stream pipelines

>  스트림 소스 --> 중간 처리 --> 최종 처리 --> 결과



```java
List<Integer> oddNumbers = numbers.stream()	                    // 스트림소스를 스트림으로 변환
                                  .filter(num -> num % 2 == 1) 	// 중간 처리(filter)
                                  .collect(toList())          	// 최종 처리(collect)
```



중간처리 종류

* 필터링 - distinct, filter
* 매핑 - map, flatMap, boxed
* 정렬 - sorted
* 루핑 - peek



최종처리 종류

* 매칭 - allMatch, anyMatch, noneMatch
* 집계 - count, findFirst, max, min, average, reduce, sum
* 루핑 - forEach
* 수집 - collect




## 사용예시

### Predicate

1~ 10까지 정수 중에서 홀수만 필터링하는 예제

```java
 int[] odds = IntStream.range(1, 10)
                       .filter(i -> i % 2 == 1)
                       .toArray();
```

odds = [1,3,5,7,9]



### Distinct

중복 제거하는 예제

```java
 int[] result = IntStream.of(1, 2, 3, 2, 4, 5)
                         .distinct()
                         .toArray();
```

 result = [1, 2, 3, 4, 5]



### TakeWhile 

java9+, predicate를 이용한 슬라이싱 방법 중 하나로

takeWhile의 인자로 넘겨준 predicate가 처음 true로 준 아이템까지 반환한다.

```java
 int[] result = IntStream.of(1, 2, 3, 4, 5, 6)
                         .takeWhile(num -> num < 4)
                         .toArray();
```

 result = [1, 2, 3]



### DropWhile

java9+,  나머지 요소를 선택하기 위해서는 dropWhile을 활용한다.

```java
 int[] result = IntStream.of(1, 2, 3, 4, 5, 6)
                         .dropWhile(num -> num < 4)
                         .toArray();
```

 result = [4, 5, 6]



### Skip, Limit

* skip : n개 요소를 제외한 스트림을 반환

* limit : 최대 요소 n개까지 반환한다.



앞의 2개 요소 건너띄고 최대 3개까지 반환하는 예시

```java
 int[] result = IntStream.of(1, 2, 3, 4, 5, 6)
                         .skip(2)
                         .limit(3)
                         .toArray();
```

 result = [3, 4, 5]



### Map 

User에서 이름만 추출하는 예시

```java
List<String> userNames = users.stream()
                              .map(user -> user.getName()) //User::getName 도 가능
                              .collect(Collectors.toList());
```



### FlatMap

여러 개의 문자열 목록을 한 글자씩 분리된 하나의 배열로 변환하기

```java
List<String> inputs = List.of("ab", "cd");
List<String> distinctStrings = inputs.stream()
                                     .map(input -> input.split("")) //["a", "b"], ["c", "d"]
  				                           .flatMap(Arrays::stream)       //"a", "b", "c", "d"
                                     .collect(Collectors.toList())  //["a", "b", "c", "d"]
```



***



### NoneMatch

주어진 predicated와 **모든 요소가 불일치**할 때만 true를 리턴한다.

```java
boolean result = IntStream.of(1, 2, 3, 4, 5, 6)
                          .noneMatch(i -> i > 10);
```

해당 경우는 모두 10보다 작으므로 result = true



`noneMatch(i -> i < 3)` 이었다면 요소 탐색시 요소 1일때 바로 불일치하므로 false를 리턴한게 된다. (short circuit)



### AllMatch

주어진 predicated와 **모든 요소가 일치**할 때만 true를 리턴한다.

```java
boolean result = IntStream.of(1, 2, 3, 4, 5, 6)
                          .allMatch(i -> i < 10);
```

해당 경우는 모두 10보다 작으므로 result = true



### AnyMatch

주어진 predicated와  **일치하는 요소가 1개라도 있으면** true를 리턴한다.

```java
boolean result = IntStream.of(1, 2, 3, 4, 5, 6)
                          .anyMatch(i -> i == 2);
```

요소 중 2인 요소가 있으므로  result = true



***



### FindFirst

스트림의 **맨 처음 요소** 리턴. 

```java
Optional<Integer> result = IntStream.of(1, 2, 3, 5, 4, 6)
                                    .filter(num -> num > 3)
                                    .mapToObj(Integer::new)
                                    .findFirst();
```

result.get()은 5이다.

필터된 요소 중 [5, 4, 3] 맨 처음 요소는 5이기 때문.



### FindAny

스트림 중 어느 한 요소를 리턴. 병렬 실행시 순서가 상관없다면 findAny를 활용

```java
Optional<Integer> result = IntStream.of(1, 2, 3, 5, 4, 6)
                                    .filter(num -> num > 3)
                                    .mapToObj(Integer::new)
                                    .findFirst();
```

result.get()는 [5, 4, 6] 중 한 개가 리턴될 것이다.





***



### Reduce

```java
 OptionalInt result = IntStream.of(1, 2, 3, 4, 5)
                .reduce((a, b) -> a + b); //Integer::sum으로 대체 가능
```

result는 15이다. 

reduce할 대상의 요소가 0개일 경우 result는 OptionalInt.empty를 반환한다.



Optional이 아니라 멱등성을 유지한 상태로 사용하고 싶다면

```java
int result = IntStream.of(1, 2, 3, 4, 5)
                      .reduce(0, (a, b) -> a + b);
```

첫번째 인자에 멱등성을 유지할 수 있게 하는 초기값을 넣어주면 된다. 

해당 예시는 덧셈이라 0으로 넣어주었다.





### Sum, Min, Max

숫자 스트림(ex. IntStream, LongStream)에 특화되어있다.

```java
int sum = IntStream.of(1, 2, 3, 4, 5)
                   .sum(); // 15

int min = IntStream.of(1, 2, 3, 4, 5)
                   .min(); // 1

int max = IntStream.of(1, 2, 3, 4, 5)
                   .max(); // 5
```



reference 타입으로 해당 계산 API를 사용하고 싶다면 언박싱이 필요하다.

```java
 List<Integer> inputs = List.of(1, 2, 3, 4, 5);
 int result = inputs.stream()
                    .mapToInt(Integer::intValue)  //unboxing 
                    .sum();
```



***







## 스트림 만들기



### 값으로 스트림 만들기

`Stream.of` 이용하여 스트림을 만들 수 있다.

```java
List<String> list = Stream.of("a", "b", "c")
                           .collect(Collectors.toList());
```



### 배열로 스트림 만들기

`Arrays.stream` 이용하여 스트림을 만들 수 있다.

```java
String[] inputs = {"a", "b", "c"};
List<String> list = Arrays.stream(inputs)
                          .collect(Collectors.toList());
```



### 무한 스트림 만들기

`iterate` 이용하여 무한 스트림(크기가 고정되지 않은 스트림)을 만들 수 있다.

보통 무한한 값을 출력하지 않도록 `limit`와 함께 사용하고는 한다.



1부터 1씩 증가하는 무한스트림을 생성 후 limit 10으로 제한하여 아이템 10개 반환

```java
Stream.iterate(1, num -> num + 1)
      .limit(10)
      .forEach(System.out::println);
```

1~10까지 출력된다.



`generate`를 활용해서도 할 수 있다. 단, generate는 `Supplier<T>`를 받아 스트림을 생성한다.

아래는 난수 5개를 출력하는 예시이다.

```java
Stream.generate(Math::randam)
      .limit(5)
      .forEach(System.out::println);
```





### Stream.ofNullable

스트림 소스가 null일 경우 Stream.empty()를 방어적으로 넣어줘야했다면

```java
Stream<String> property = value == null ? Stream.empty() : Stream.of(value);
```



자바9부터는 `Stream.ofNullable`을 활용해서 해당 케이스를 커버할 수 있다.

```java
Stream<String> property = Stream.ofNullable(value);
```
