Wrapper to capture key and value types of consumed records. Why is this necessary? Refer to:

https://docs.oracle.com/javase/tutorial/java/generics/capture.html

```java
class ConsumeRecordHelper {
  <K> void fnConsumeRecordHelper(ConsumerRecord<K,C> record) {
    logger.debug("Consuming record: {}", record);
    consumerFunction.accept(record.value());
  }
}

ConsumeRecordHelper helper = new ConsumeRecordHelper();

lstListenerThreads.add(
    listenerThreadFactory.newThread(() -> { 
      consumer.start(lstTopics, helper::fnConsumeRecordHelper); 
    })
);
```