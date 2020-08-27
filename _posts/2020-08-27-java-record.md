---
layout: post
title: Java Record
tag: jdk14
---



#### Record란?

아래 기능을 제공해주는 데이터 클래스이며 `recode` 키워드를 사용한다.

또한, 필드 값은 변경 불가능하므로 불변 객체를 만들 때 사용하면 된다.

* `toString()`
* `hashCode()`와 `equals()`
* getter methods
* A public constructor



>Record기능은 현재 jdk14, preview로 제공되므로 테스트를 해보려면 아래와 같은 옵션이 필요하다
>
>`--enable-preview`



그동안 x, y좌표를 표현하는 Point객체를 만들기 위해서는 아래와 같이 작성해야했다.

```java
public class Point {
    private final int x;
    private final int y;
 
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
 
    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Point point = (Point) o;
        return x == point.x && y == point.y;
    }
 
    @Override
    public String toString() {
        return "Point [x=" + x + ", y=" + y + "]";
    }

    public String getX() {
        return x;
    }
  
    public String getY() {
        return y;
    }
}
```



그 동안 hashCode, equals, getter등 보일러 플레이트 코드처럼 작성해왔다. 물론, IDE의 generate code 기능이나 lombok을 사용하는 식으로 우리의 소중한 노동력을 줄일 수는 있었다. 



하지만 jdk14에 새로 나온  `record`를 이용하면 아래와 같이 표현할 수 있다.

```java
public record Point (int x, int y) {}
```

장황하던 표현이 `record`키워드로 축약되니 보기 편해졌다(편안)



`Point.class` 파일을 decompile해보니 아래와 같이 컴파일러가 자동 생성해줌을 알 수 있다

~~이것이 언어가 제공해주는 편리함~~

```java
public final class Point extends java.lang.Record {
    private final int x;
    private final int y;

    public Point(int x, int y) { /* compiled code */ }

    public java.lang.String toString() { /* compiled code */ }

    public final int hashCode() { /* compiled code */ }

    public final boolean equals(java.lang.Object o) { /* compiled code */ }

    public int x() { /* compiled code */ }

    public int y() { /* compiled code */ }
}
```



---

### Record

```java
public record Point (int x, int y) {}
```



#### equals

```java
Point a = new Point(0, 0);
Point b = new Point(0, 0);
Point c = new Point(0, 5);

System.out.println(a.equals(b)); //true
System.out.println(a.equals(c)); //false
```



#### toString

```java
Point p = new Point(5, 10);

System.out.println(p.toString()); 
```



아래와 같은 형식으로 출력된다.

```
Point[x=5, y=10]
```



#### getter

getXXX()패턴이 아닌 필드명으로 호출하면 된다

* ex. `getX()` --> `x()`

```java
Point p = new Point(5, 10);

System.out.println("x : " + p.x());  // 5
```



#### Constructor

모든 필드를 가지는 생성자 외에 다른 새 생성자도 만들 수 있다.

```java
public record Point (int x, int y) {
   public Persion(int x) {
      this.(x, 0);
   }
}
```





#### reference

* https://www.baeldung.com/java-record-keyword
* https://blogs.oracle.com/javamagazine/records-come-to-java



