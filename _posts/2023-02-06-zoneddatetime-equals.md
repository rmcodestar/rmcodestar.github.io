---
layout: post
title:  ZonedDateTime equals vs isEqual
category: Java
tag: [Java, ZonedDateTime]
---

### ZonedDateTime 이해하기
`ZonedDateTime`은 java8부터 java.time 패키지에 추가된 타임존과 시차를 적용하여 날짜와 시간을 계산할 수 있는 클래스입니다.


2024-01-01T00:00:00+09:00을 표현한다고 할 때 서울 타임존을 이용해서는 `ZoneId.of("Asia/Seoul")`을 이용할 수 있고,
ZoneOffset을 이용해서 KST는 UTC+09이므로 `ZoneOffset.of("+09:00")`을 이용해서 계산할 수 있습니다.
```java
var today = LocalDate.of(2024, 1, 1).atTime(0, 0, 0);

//Timezone
ZonedDateTime.of(today, ZoneId.of("Asia/Seoul"))

//ZoneOffset
ZonedDateTime.of(today, ZoneOffset.of("+09:00"))
```

<br>

### `equals` vs `isEqual`
두 개의 `ZonedDateTime`을 비교하다가 실수를 한 부분이 있었습니다. 바로 시간 비교였는데요.

시간 비교시 `equals`와 `isEqual` 메서드의 판단이 각각 다르다는 것을 뒤늦게 알았습니다.

```java
//java.time.ZonedDateTime
@Override
public boolean equals(Object obj) {
    if (this == obj) {
        return true;
    }
    if (obj instanceof ZonedDateTime) {
        ZonedDateTime other = (ZonedDateTime) obj;
        return dateTime.equals(other.dateTime) &&
            offset.equals(other.offset) &&
            zone.equals(other.zone);
    }
    return false;
}

default boolean isEqual(ChronoZonedDateTime<?> other) {
    return toEpochSecond() == other.toEpochSecond() &&
            toLocalTime().getNano() == other.toLocalTime().getNano();
}
```

차이점을 아시겠나요?

`equals`의 경우 ZonedDateTime의 구성요소인 `LocalDateTime`, `ZoneOffset`, `ZoneId`가 모두 일치해야 같다고 판단합니다.

`isEqual`의 경우 Ephoch 시간을 가지고만 비교를 합니다.

> epoch란 1970년 1월 1일 00:00:00 협정 세계시(UTC) 부터의 경과 시간을 초로 환산하여 정수로 나타낸 값

```java
@Test
public void testcase1() {
    var today = LocalDate.of(2024, 1, 1).atTime(0, 0, 0);
    var zonedDateTime1 = ZonedDateTime.of(today, ZoneOffset.of("+09:00")); //ZoneId=+09:00
    var zonedDateTime2 = ZonedDateTime.of(today, ZoneId.of("Asia/Seoul")); //ZoneId=Asia/Seoul

    assertEquals(zonedDateTime1.equals(zonedDateTime2), false); //Ephoch 시간이 같더라도 zoneId가 다르기 때문에 다르다고 판단
    assertEquals(zonedDateTime1.isEqual(zonedDateTime2), true);
}
```
같은 Ephoch 시간인지를 판단하고자 하려면 `isEqual`을 쓰는 것이 의도가 맞습니다.

아니면 `Instant`객체로 변환하거나, 동일한 `ZoneId`로 변환한 후 `equals`를 쓰는 것도 방법이 될 수 있습니다.

```java
@Test
public void testcase2() {
    var today = LocalDate.of(2024, 1, 1).atTime(0, 0, 0);
    var zonedDateTime1 = ZonedDateTime.of(today, ZoneOffset.of("+09:00"));
    var zonedDateTime2 = ZonedDateTime.of(today, ZoneId.of("Asia/Seoul"));

    assertEquals(zonedDateTime1.toInstant().equals(zonedDateTime2.toInstant()), true);
    assertEquals(zonedDateTime1.withZoneSameInstant(zonedDateTime2.getZone()).equals(zonedDateTime2), true);
}
```

