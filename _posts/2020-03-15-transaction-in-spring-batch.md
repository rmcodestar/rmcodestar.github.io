---
layout: post
title:  transaction in spring batch
category: spring
tag: [spring, spring batch]
---



## chunk processing flow

![img](https://blog.codecentric.de/files/2012/03/Blog_Transactions_Listeners-1024x528.png)



## Listeners
- JobExecutionListener
  - beforeJob
  - afterJob
- StepExecutionListener
  - beforeStep
  - afterStep
- ChunkListener
  - beforeChunk - 청크 트랜잭션 내부에서 실행
  - afterChunk - 청크 트랜잭션 외부에서 실행
- ItemReadListener -  모두 청크 트랜잭션 내에서 실행
  - beforeRead
  - afterRead
  - onReadError
- ItemProcessListener - 모두 청크 트랜잭션 내에서 실행
  - beforeProcess
  - afterProcess
  - onProcessError
- ItemWriteListener - 모두 청크 트랜잭션 내에서 실행
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





## 소스보기

청크 단위의 작업이 일어나는 순서나 listener들이 호출되는 것을 소스단에서 확인하기 위해서 아래 소스부터 보면 좋다

`TaskletStep.doExecute` (spring-batch-core-4.2-1-RELEASE 소스 기준)

```java
package org.springframework.batch.core.step.tasklet;

public class TaskletStep extends AbstractStep {
  protected void doExecute(StepExecution stepExecution) throws Exception {

    ...생략...

		stepOperations.iterate(new StepContextRepeatCallback(stepExecution) {

			@Override
			public RepeatStatus doInChunkContext(RepeatContext repeatContext, ChunkContext chunkContext)
					throws Exception {

				StepExecution stepExecution = chunkContext.getStepContext().getStepExecution();

				// Before starting a new transaction, check for
				// interruption.
				interruptionPolicy.checkInterrupted(stepExecution);

				RepeatStatus result;
				try {
					result = new TransactionTemplate(transactionManager, transactionAttribute)
					.execute(new ChunkTransactionCallback(chunkContext, semaphore));
				}
				catch (UncheckedTransactionException e) {
					// Allow checked exceptions to be thrown inside callback
					throw (Exception) e.getCause();
				}

				chunkListener.afterChunk(chunkContext);

				// Check for interruption after transaction as well, so that
				// the interrupted exception is correctly propagated up to
				// caller
				interruptionPolicy.checkInterrupted(stepExecution);

				return result == null ? RepeatStatus.FINISHED : result;
			}

		});

	}
```



```java
package org.springframework.transaction.support;

public class TransactionTemplate extends DefaultTransactionDefinition implements TransactionOperations, InitializingBean {

public <T> T execute(TransactionCallback<T> action) throws TransactionException {
        Assert.state(this.transactionManager != null, "No PlatformTransactionManager set");
        if (this.transactionManager instanceof CallbackPreferringPlatformTransactionManager) {
            return ((CallbackPreferringPlatformTransactionManager)this.transactionManager).execute(this, action);
        } else {
            TransactionStatus status = this.transactionManager.getTransaction(this);

            Object result;
            try {
                result = action.doInTransaction(status);
            } catch (Error | RuntimeException var5) {
                this.rollbackOnException(status, var5);
                throw var5;
            } catch (Throwable var6) {
                this.rollbackOnException(status, var6);
                throw new UndeclaredThrowableException(var6, "TransactionCallback threw undeclared checked exception");
            }

            this.transactionManager.commit(status);
            return result;
        }
    }
```



## Reference

* https://blog.codecentric.de/en/2012/03/transactions-in-spring-batch-part-1-the-basics/
* https://blog.codecentric.de/en/2012/03/transactions-in-spring-batch-part-2-restart-cursor-based-reading-and-listeners/
