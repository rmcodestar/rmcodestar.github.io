---
layout: post
title:  Transaction in spring batch
category: Spring
tag: [Spring, Spring batch]
---
## Chunk processing flow
![img](https://www.codecentric.de/_next/image?url=https%3A%2F%2Fmedia.graphassets.com%2Foutput%3Dformat%3Awebp%2F2dOPPFENSg65r7ZfsEo3&w=1920&q=75)

## Listeners
- JobExecutionListener  `#AbstractJob`
  - beforeJob
  - afterJob
- StepExecutionListener `#AbstractStep`
  - beforeStep
  - afterStep
- ChunkListener `#TaskletStep`
  - beforeChunk - 청크 트랜잭션 내부에서 실행
  - afterChunk - 청크 트랜잭션 외부에서 실행
- ItemReadListener -  모두 청크 트랜잭션 내에서 실행 `#ChunkProcessor`
  - beforeRead
  - afterRead
  - onReadError
- ItemProcessListener - 모두 청크 트랜잭션 내에서 실행 `#ChunkProcessor`
  - beforeProcess
  - afterProcess
  - onProcessError
- ItemWriteListener - 모두 청크 트랜잭션 내에서 실행 `#ChunkProcessor`
  - beforeWrite 
  - afterWrite
  - onWriteError



## 각 실행되는 시점

정상적으로 실행된 경우의 시나리오

1. beforeJob
2. beforeStep
3. **transaction.getTransaction** 
4. beforeChunk
5. Chunk processing (read, process, write)
6. **transaction.commit**
7. afterChunk
8. afterStep
9. afterJob



특정 청크에서 에러가 발생했을 때의 시나리오

1. beforeJob
2. beforeStep
3. **transaction.getTransaction** 
4. beforeChunk
5. exception in chunk processing (read, process, write)
6. **발생한 chunk는 transaction.rollback**
7. afterChunkError
8. (retry로직이 있을 경우) 4번의 에러 발생한 chunk 아이템 모두 다시 chunk processing 실행됨
9. afterStep
10. afterJob


## Reference
* https://docs.spring.io/spring-batch/docs/current-SNAPSHOT/reference/html/index.html
* https://www.codecentric.de/wissens-hub/blog/transactions-in-spring-batch-part-1-the-basics
* https://www.codecentric.de/wissens-hub/blog/transactions-in-spring-batch-part-2-restart-cursor-based-reading-and-listeners
* https://www.codecentric.de/wissens-hub/blog/transactions-in-spring-batch-part-3-skip-and-retry
