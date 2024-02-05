---
layout: post
title: Spring batch skip과 processorTransactional
category: Spring
tag: [Spring, Spring batch, processorTransactional]
---

### Spring batch에서의 skip 기능에 대한 오해
spring batch 에서 특정 exception 발생시 발생한 아이템을 skip할 수 있는 기능이 있다.

바로 아래처럼 사용할 수 있는데 테스트하다가 내 예상과 다르게 동작하는 부분이 있었다.

<br>

**테스트 코드**

```java
@Bean
public Step testStep() {
    return stepBuilderFactory.get("testStep")
            .<BatchRecord, BatchRecord>chunk(3)
            .faultTolerant()
            .skip(IllegalArgumentException.class) //IllegalArgumentException 발생 시 skip함
            .skipLimit(3)  //최대 skip은 3번까지 허용
            .reader(batchRecordReader())
            .processor(batchRecordProcessor())
            .writer(batchRecordJpaItemWriter())
            .build();
}
```


<br>


**테스트 시나리오**

총 아이템은 10개가 있고 chunk size는 3개이다

> Items = [1,2,3,4,5,6,7,8,9,10]
>
> Chunks = [1,2,3], [4,5,6], [7,8,9], [10]

<br>

총 4번의 청크가 생기며 아이템 5가 processor에서 처리될 때 `IllegalArgumentException`이 발생된다고 하자.

(나머지는 정상 처리)

<br>

**processor 테스트 코드**

```java
public ItemProcessor<BatchRecord, BatchRecord> batchRecordProcessor() {
    return batchRecord -> {
        log.info("[PROCESSOR] {}", batchRecord);

        if (batchRecord.getId() == 5) { ///failed case
            throw new IllegalArgumentException("test");
        }

        batchRecord.setStatus(BatchStatus.DONE);
        return batchRecord;
    };
}
```

<br>

내 예상에는 [1,2,3] [4,5(에러 발생), 6], [7,8,9], [10] 이렇게 processor에서 처리될 줄 알았다.(writer에서도 동일)

하지만 예상과 다르게 [1,2,3], [4, 5(에러 발생)], [4, 6], [7, 8, 9], [10] 이렇게 processor 아이템이 처리되는 것이 아닌가?

<br>


> expected : [1,2,3] [4,5(에러 발생), 6], [7,8,9], [10]
>
> actual : [1,2,3], [4, 5(에러 발생)], [4, 6], [7, 8, 9], [10] 

<br>


에러가 발생했던 chunk에서 이미 처리되었던 아이템 4가 processor에서 재처리가 되고 있었다.

아이템 4은 총 2번 처리가 된 것이다.

<br>

**관련 로그**

```
Job: [SimpleJob: [name=sampleJob]] launched
Executing step: [testStep]
[STEP-before]

[CHUNK-before]
[PROCESSOR] BatchRecord(id=1)
[PROCESSOR] BatchRecord(id=2)
[PROCESSOR] BatchRecord(id=3)
[CHUNK-after] ItemCount: 3

[CHUNK-before]
[PROCESSOR] BatchRecord(id=4)
[PROCESSOR] BatchRecord(id=5)
[CHUNK-after-error] summary: StepExecution: id=1, version=2, name=testStep, status=STARTED, exitStatus=EXECUTING, readCount=6, filterCount=0, writeCount=3 readSkipCount=0, writeSkipCount=0, processSkipCount=0, commitCount=1, rollbackCount=1

[CHUNK-before]
[PROCESSOR] BatchRecord(id=4)
[PROCESSOR] BatchRecord(id=6)
[CHUNK-after] ItemCount: 6

[CHUNK-before]
[PROCESSOR] BatchRecord(id=7)
[PROCESSOR] BatchRecord(id=8)
[PROCESSOR] BatchRecord(id=9)
[CHUNK-after] ItemCount: 9

[CHUNK-before]
[PROCESSOR] BatchRecord(id=10)
[CHUNK-after] ItemCount: 10

[STEP-after] readCount : 10, writeCount : 9, skipCount : 1, commitCount : 4 
```

<br>


### 왜 이렇게 동작하는 것일까? 🤔

chunk내에서 exception이 발생하면 transaction rollback을 하고 다시 processor-writer를 수행하게 된다.

processor안에서도 db작업을 했을 수도 있기 때문에 다시 수행하는 로직에 processor도 포함이 되어있는 것이다.

![transaction](https://www.codecentric.de/_next/image?url=https%3A%2F%2Fmedia.graphassets.com%2Foutput%3Dformat%3Awebp%2F2dOPPFENSg65r7ZfsEo3&w=1920&q=75)

<br>


### 해결방법: processorTransactional 적용 💡

`FaultTolerantChunkProcessor` 에서 아래 로직이 수행되고

```java
O output = batchRetryTemplate.execute(retryCallback, recoveryCallback, new DefaultRetryState(getInputKey(item), rollbackClassifier));
```



`retryCallback`안에 로직을 보면 `processorTransactional`이 false일 때 processor가 처리한 아이템을 캐싱하는 것이 보일 것이다. 

만약 processor 안 로직에 transaction이 필요한 작업이라면 (예를 들면 db작업) 결과를 캐싱을 하면 안되겠지만

그게 아니라면 Processor 결과를 캐싱해두면 재시도하고자 하는 chunk의 아이템을 재처리하지 않아도 되게 된다.

```java
RetryCallback<O, Exception> retryCallback = new RetryCallback<O, Exception>() {
    @Override
    public O doWithRetry(RetryContext context) throws Exception {
        O output = null;
        try {
            count.incrementAndGet();
            O cached = (cacheIterator != null && cacheIterator.hasNext()) ? cacheIterator.next() : null;
            if (cached != null && !processorTransactional) {
                output = cached;
            }
            else {
                output = doProcess(item);
                if (output == null) {
                    data.incrementFilterCount();
                } else if (!processorTransactional && !data.scanning()) {
                    cache.add(output);
                }
            }
    
            ...생략...
    }
}
```

<br>

 **테스트 코드** 

`processorNonTransactional()` 를 추가하고 다시 돌려보면

```java
@Bean
public Step testStep() {
    return stepBuilderFactory.get("testStep")
            .<BatchRecord, BatchRecord>chunk(3)
            .faultTolerant()
            .skip(IllegalArgumentException.class)
            .skipLimit(3)
            .processorNonTransactional()  // <--- NEW
            .reader(batchRecordReader())
            .processor(batchRecordProcessor())
            .writer(batchRecordJpaItemWriter())
            .transactionManager(new ResourcelessTransactionManager())
            .build();
}
```

<br>

[1, 2, 3], [4, 5(에러 발생)], [6], [7,8,9], [10] 이렇게 처리되게 된다.

<br>

**결과 로그**

```
Executing step: [testStep]
[STEP-before]
[CHUNK-before]
[PROCESSOR] BatchRecord(id=1)
[PROCESSOR] BatchRecord(id=2)
[PROCESSOR] BatchRecord(id=3)
[CHUNK-after] ItemCount: 3

[CHUNK-before]
[PROCESSOR] BatchRecord(id=4)
[PROCESSOR] BatchRecord(id=5)
[CHUNK-after-error] summary: StepExecution: id=1, version=2, name=testStep, status=STARTED, exitStatus=EXECUTING, readCount=6, filterCount=0, writeCount=3 readSkipCount=0, writeSkipCount=0, processSkipCount=0, commitCount=1, rollbackCount=1

[CHUNK-before]
[PROCESSOR] BatchRecord(id=6)
[CHUNK-after] ItemCount: 6

[CHUNK-before]
[PROCESSOR] BatchRecord(id=7)
[PROCESSOR] BatchRecord(id=8)
[PROCESSOR] BatchRecord(id=9)
[CHUNK-after] ItemCount: 9

[CHUNK-before]
[PROCESSOR] BatchRecord(id=10)
[CHUNK-after] ItemCount: 10

[STEP-after] readCount : 10, writeCount : 9, skipCount : 1, commitCount : 4
```



#### Reference
- https://blog.codecentric.de/en/2012/03/transactions-in-spring-batch-part-3-skip-and-retry
