# Passing segment context between threads in a multithreaded application<a name="xray-sdk-java-multithreading"></a>

When you create a new thread in your application, the `AWSXRayRecorder` doesn't maintain a reference to the current segment or subsegment [Entity](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Entity.html)\. If you use an instrumented client in the new thread, the SDK tries to write to a segment that doesn't exist, causing a [SegmentNotFoundException](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/exceptions/SegmentNotFoundException.html)\.

To avoid throwing exceptions during development, you can configure the recorder with a [ContextMissingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/ContextMissingStrategy.html) that tells it to log an error instead\. You can configure the strategy in code with [SetContextMissingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html#setContextMissingStrategy(com.amazonaws.xray.strategy.ContextMissingStrategy)), or configure equivalent options with an [environment variable](xray-sdk-java-configuration.md#xray-sdk-java-configuration-envvars) or [system property](xray-sdk-java-configuration.md#xray-sdk-java-configuration-sysprops)\.

One way to address the error is to use a new segment by calling [beginSegment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html#beginSegment(java.lang.String)) when you start the thread and [endSegment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html#endSegment()) when you close it\. This works if you are instrumenting code that doesn't run in response to an HTTP request, like code that runs when your application starts\.

If you use multiple threads to handle incoming requests, you can pass the current segment or subsegment to the new thread and provide it to the global recorder\. This ensures that the information recorded within the new thread is associated with the same segment as the rest of the information recorded about that request\. Once the segment is available in the new thread, you can execute any runnable with access to that segment's context using the `segment.run(() -> { ... })` [method](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Entity.html#run(java.lang.Runnable)).

See [Using instrumented clients in worker threads](scorekeep-workerthreads.md) for an example\.

## Using X-Ray with Asynchronous Programming<a name="using-asynchronous-programming"></a>

The X-Ray SDK for Java can be used in asynchronous Java programs with [SegmentContextExecutors](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/contexts/SegmentContextExecutors.html). The SegmentContextExecutor implements the Executor interface, which means it can be passed into all asynchronous operations of a [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html). Then any asynchronous operations will be executed with the correct segment in its context.

**Example App.java - Passing SegmentContextExecutor to CompletableFuture**

```
DynamoDbAsyncClient client = DynamoDbAsyncClient.create();

AWSXRay.beginSegment();

// ...

client.getItem(request).thenComposeAsync(response -> {
    // If we did not provide the segment context executor, this request would not be traced correctly.
    return client.getItem(request2);
}, SegmentContextExecutors.newSegmentContextExecutor());
```

