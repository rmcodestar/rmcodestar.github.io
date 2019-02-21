---
layout: post
title: 자바 람다와 스트림
category: JAVA
tag: [lambda, stream]
---
# 람다

<br>

### 익명클래스의 인스턴스 대신 람다를 사용하자

[AS-IS] 익명클래스의 인스턴스 사용

```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    } 
});
```

<br>


[TO-BE] 람다 사용

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

<br>


> 타입을 명시해야 코드가 더 명확할 때만 타입을 명시하고,
>
> 람다의 모든 매개변수 타입은 생략하자.(타입 추론)


<br>



### enum을 이용한 상수별로 다르게 구현해야 하던 것을 람다로 대체할 수 있다

[AS-IS] enum 사용 예제

```java
public enum Operation {
    PLUS("+") {
        public double calculate(double x, double y) {
            return x + y;
        }
    }
    , MINUS("-") {
        public double calculate(double x, double y) {
            return x - y;
        }
    };
    
    private final String symbol;
    
    Operation (String symbol) {
        this.symbol = symbol;
    }

    public abstract double calculate(double x, double y);
}
```

<br>


[TO-BE] 람다 사용

```java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y);

    private final String symbol;
    public final DoubleBinaryOperator function;

    Operation(String symbol, DoubleBinaryOperator function) {
        this.symbol = symbol;
        this.function = function;
    }

    public double calculate(double d1, double d2) {
        return this.function.applyAsDouble(d1, d2);
    }
}
```

`abstract method` 구현체 대신 `DoubleBinaryOperator` 람다를 저장해두고 `calculate`메서드에서 람다를 호출하자.


<br>


```java
Operation.PLUS.calculate(1, 2)
```

<br>


[참고] `java.util.function.DoubleBinaryOperator`

```java
@FunctionalInterface
public interface DoubleBinaryOperator {
    double applyAsDouble(double left, double right);
}
```

<br>


간단하게 세 줄안에 끝나는 로직일 경우, 람다를 쓰면 가독성이 좋지만 그 이상 로직일 경우엔 기존의 방법을 쓰는 것이 낫다.

또한 람다는 이름이 없고 문서화도 하지 못하는 점을 주의하자.


<br>



### 람다가 대체할 수 없는때

* `this` 키워드 용도가 다를 때, 함수 객체가 자신을 참조해야 할 때는 익명클래스를 사용

  > 람다에서의 `this`는 바깥 인스턴스를 가리키지만 익명클래스의 `this`는 자기 자신을 가리킨다.

* 직렬화해야만 하는 함수 객체라면 비추천

  > 람다도 익명클래스처럼 직렬화 형태가 구현별로 다를 수 있다.



<br>


### 메서드 참조는 람다의 간단명료한 대안이 될 수 있다

[AS-IS] 람다 사용

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

<br>


[TO-BE] 메서드 참조(method reference)

```java
map.merge(key, 1, Integer::sum)
```

<br>


[참고] `java.util.Map`

```java
V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction)
```

<br>


또 다른 예시

| 메서드 참조 유형   | 람다                                                     | 메서드 참조            |
| ------------------ | -------------------------------------------------------- | ---------------------- |
| 정적               | str -> Integer.parseInt(str)                             | Integer::parseInt      |
| 한정적(인스턴스)   | Instant then = Instant.now();<br />t -> then.isAfter(t); | Instant.now()::isAfter |
| 비한정적(인스턴스) | str -> str.toLowerCase()                                 | String::toLowerCase    |
| 클래스 생성자      | () -> new TreeMap<K, V>()                                | TreeMap<K, V>::new     |
| 배열 생성자        | len -> new int[len]                                      | int[]::new             |


<br>



### 표준 함수형 인터페이스를 사용하라

>  필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라
>

<br>


java.util.function 패키지에는 43 개의 인터페이스가 담겨 있다.

기존 6개의 인터페이스만 기억하면 나머지는 유추해서 사용하기 쉽다.

<br>


**[기본 인터페이스]**

| 인터페이스        | 함수 시그니처        | 예시                |
| ----------------- | -------------------- | ------------------- |
| UnaryOperator<T>  | T apply (T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply (T t1, T t2) | BinInteger::add     |
| Predicate<T>      | boolean test (T t)   | Collection::isEmpty |
| Function <T,>     | R apply (T t)        | Arrays::asList      |
| Supplier<T>       | T get()              | Instant::now        |
| Consumer<T>       | void accept (T t)    | System.out.println  |



* 기본 인터페이스는 primitive 타입인 `int`, `double`, `long`용으로 각 변형이 존재한다.

  > `int`를 받는 `Predicate`는 `IntPredicate`, `long`이면 `LongPredicate`가 된다.
  >
  > reference 타입이면 그냥 기본 인터페이스가 제네릭이니까 그걸 쓰면 된다.ㅎ

* `Function` 인터페이스에는 기본 타입을 반환하는 변형이 총 9개 더 있다.

  > 입력과 결과 타입이 모두 기본 타입이면 접두어로 `{Src}To{Result}`를 사용한다.
  >
  > `long`을 받아 `int`로 받환하면 `LongToIntFunction`이다

* 기본 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있다.
  * `BiPredicate<T, U>`
  * `BiFunction<T, R>`
  * `BiConsumer<T, R>`

* `BooleanSupplier`인터페이스는 `boolean`을 받환하도록 한 `Supplier`의 변형이다



<br>

***


## 스트림

<br>

### 스트림은 주의해서 사용하라

* 스프림 파이프라인은 지연평가다(lazy evaluation). 종단 연산을 빼먹는 일이 없도록 하자
*  스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.
* 기본 타입인 char용 스트림을 지원하지 않는다.
* `forEach` 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.
* 스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다.

<br>

### 자바와 동시성 프로그래밍

* 스레드, 동기화, wait/notify를 지원

* 자바 5 : `java.util.concurrent` 라이브러리와 실행자(`Executor`) 프레임워크 지원
* 자바 7 : 고성능 병렬 분해 프레임 워크인 포크-조인(`fork-join`) 패키지 추가
* 자바 8 : 병렬로 실행할 수 있는 스트림 지원, `parallel`

<br>

### 스트림 파이프라인 병렬화

* 데이터 소스가 `Stream.iterate`거나 `limit`을 쓰면 성능을 기대하기 어렵다.
* 병렬화의 효과가 좋았던 경우
  * 대체로 스트림의 소스가 `ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap`의 인스턴스일 때
  * 배열, int 범위, long 범위일 때
* 종단 연산 중 병렬화에 가장 적합한 것은
  *  축소(reduction)이다 `reduce`, `max`, `min`, `sum`
  * 조건에 맞으면 바로 반환된는 메서드도 적합. `anyMatch`, `allMatch`, `noneMatch`
* 계산도 올바로 수행하고 성능도 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화를 사용하지 말자
