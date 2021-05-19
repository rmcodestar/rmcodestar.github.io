---
layout: post
title:  SchemaBuilder로 avro 스키마 생성하기
category: avro
tag: [kafka, avro]
---



## Backgroud

[Guide to Apache Avro- baeldung](https://www.baeldung.com/java-apache-avro)

<br>

## SchemaBuilder로 avro 스키마 생성하기

json으로 노가다를 하기 싫었던 나는 SchemaBuilder를 이용하기로 마음먹었다.

근데 nullable record 표현 및.. optional field와 array표현등.. 삽질을 많이 했고.. 예시로 정리하여 훗날의 나에게 도움이 되길 바라며 글로 정리해보았다.


<br>

### Example

예시를 들기 위해서 책 주문에 대한 스키마를 만들어보자 (예시를 위해 대충 만듦 주의)


BookOrder

| attribute | -         | type      | nullable | desc |
| --------- | :-------- | --------- | -------- | ---- |
| orderId   |           | Long | X        | 주문 id |
| memberNo  |           | Long | X        | 회원번호 |
| orderDate |           | String    | X        | yyyy-MM-dd |
| couponId |           | Long | O        | 사용한 쿠폰 id |
| books     |           | Objects[] | X        | 주문한 책 목록 |
|           | id        | Long | X        | 책 id |
|           | price     | Long | X        | 가격 |
|           | quantity  | Integer | X        | 수량 |
| gifts     |           | Objects[] | O        | 사은품 목록 |
|           | id        | Long | X       | 사은품 id |
|           | quantity | Integer | X        | 수량 |



Java

```java
Schema book = SchemaBuilder.record("Book")
        .namespace("com.example.kafka")
        .fields()
        .requiredLong("id")
        .requiredLong("price")
        .requiredInt("quantity")
        .endRecord();

Schema gift = SchemaBuilder.nullable().record("Gift")
        .namespace("com.example.kafka")
        .fields()
        .requiredLong("id")
        .requiredInt("quantity")
        .endRecord();

Schema bookOrder = SchemaBuilder.record("BookOrder")
        .namespace("com.example.kafka")
        .fields()
        .requiredLong("orderId")
        .requiredLong("memberNo")
        .requiredString("orderDate")
        .optionalLong("couponId")
        .name("books")
        .type()
        .array()
        .items()
        .type(book)
        .noDefault()
        .name("gifts")
        .type()
        .array()
        .items()
        .type(gift)
        .noDefault()
        .endRecord();
```

위 `bookOrder`를 `toString`으로 출력하면 아래와 같이 avro json을 쉽게 얻을 수 있다.

<br>

json

```json
{
  "type" : "record",
  "name" : "BookOrder",
  "namespace" : "com.example.kafka",
  "fields" : [ {
    "name" : "orderId",
    "type" : "long"
  }, {
    "name" : "memberNo",
    "type" : "long"
  }, {
    "name" : "orderDate",
    "type" : "string"
  }, {
    "name" : "couponId",
    "type" : [ "null", "long" ],
    "default" : null
  }, {
    "name" : "books",
    "type" : {
      "type" : "array",
      "items" : {
        "type" : "record",
        "name" : "Book",
        "fields" : [ {
          "name" : "id",
          "type" : "long"
        }, {
          "name" : "price",
          "type" : "long"
        }, {
          "name" : "quantity",
          "type" : "int"
        } ]
      }
    }
  }, {
    "name" : "gifts",
    "type" : {
      "type" : "array",
      "items" : [ {
        "type" : "record",
        "name" : "Gift",
        "fields" : [ {
          "name" : "id",
          "type" : "long"
        }, {
          "name" : "quantity",
          "type" : "int"
        } ]
      }, "null" ]
    }
  } ]
}
```

<br>



### How to

#### Record type 생성 

`SchemaBuilder.record`이용한다

혹은, nullable record를 생성하기 위핸 `SchemaBuilder.nullable().record`사용

하위 레코드에 동일한 namespace를 설정하지 않으면 `namepspace: ""`로 출력되기도 하니 주의하자.

**example**

```java
Schema testRecord = SchemaBuilder.record("TestRecord")
                .namespace("com.example.kafka") //namespace 설정
                .fields()
                .requiredLong("testField") 
                .endRecord();

Schema nullableTestRecord = SchemaBuilder.nullable.record("NullableTestRecord")
                .namespace("com.example.kafka") //namespace 설정
                .fields()
                .requiredLong("testField") 
                .endRecord();
```

```json
{
  "type" : "record",
  "name" : "TestRecord",
  "namespace" : "com.example.kafka",
  "fields" : [ {
    "name" : "testField",
    "type" : "long"
  }]
}


[ {
  "type" : "record",
  "name" : "NullableTestRecord",
  "namespace" : "com.example.kafka",
  "fields" : [ {
    "name" : "testField",
    "type" : "long"
  } ]
}, "null" ]
```

<br>


#### Primative type 표현을 위한 required\*시리즈, optional\* 시리즈 ,  nullable*시리즈 사용

**example**

```java
Schema test = SchemaBuilder.record("TestRecord")
                .namespace("com.example.kafka")
                .fields()
                .nullableLong("testField1", 0L) //type null 추가 및 default 값 설정 가능
                .requiredLong("testField2") // not nullable
                .optionalLong("testField3") // type null 추가 및 default value null 설정
                .endRecord();
```

```json
{
  "type" : "record",
  "name" : "TestRecord",
  "namespace" : "com.example.kafka",
  "fields" : [ {
    "name" : "testField1",
    "type" : [ "long", "null" ],
    "default" : 0
  }, {
    "name" : "testField2",
    "type" : "long"
  }, {
    "name" : "testField3",
    "type" : [ "null", "long" ],
    "default" : null
  } ]
}

```

<br>


#### Array 표현
**example**
```java
SchemaBuilder.record("TestRecord")
             .namespace("com.example.kafka")
             .fields()
             .name("items") // array field
             .type()
             .array()
             .items()
             .stringType()  // array item의 type 지정
             .noDefault() //array default 값 지정
             .endRecord();
```

```json
{
  "type" : "record",
  "name" : "TestRecord",
  "namespace" : "com.example.kafka",
  "fields" : [ {
    "name" : "items",
    "type" : {
      "type" : "array",
      "items" : "string"
    }
  } ]
}
```


