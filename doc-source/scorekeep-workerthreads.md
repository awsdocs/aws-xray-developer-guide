# Using Instrumented Clients in Worker Threads<a name="scorekeep-workerthreads"></a>

Scorekeep uses a worker thread to publish a notification to Amazon SNS when a user wins a game\. Publishing the notification takes longer than the rest of the request operations combined, and doesn't affect the client or user\. Therefore, performing the task asynchronously is a good way to improve response time\.

However, the X\-Ray SDK for Java doesn't know which segment was active when the thread is created\. As a result, when you try to use the instrumented AWS SDK for Java client within the thread, it throws a `SegmentNotFoundException`, crashing the thread\.

**Example web\-1\.error\.log**  

```
Exception in thread "Thread-2" com.amazonaws.xray.exceptions.SegmentNotFoundException: Failed to begin subsegment named 'AmazonSNS': segment cannot be found.
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
...
```

To fix this, the application uses `GetTraceEntity` to get a reference to the segment in the main thread, and `SetTraceEntity` to pass the segment back to the recorder in the worker thread\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/MoveFactory.java#L70](https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/MoveFactory.java#L70) – Passing trace context to a worker thread**  

```
import [com\.amazonaws\.xray\.AWSXRay](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.AWSXRayRecorder](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html);
import [com\.amazonaws\.xray\.entities\.Entity](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Entity.html);
import [com\.amazonaws\.xray\.entities\.Segment](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Segment.html);
import [com\.amazonaws\.xray\.entities\.Subsegment](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Subsegment.html);
...
      Entity segment = recorder.getTraceEntity();
      Thread comm = new Thread() {
        public void run() {
          recorder.setTraceEntity(segment);
          Subsegment subsegment = AWSXRay.beginSubsegment("## Send notification");
          Sns.sendNotification("Scorekeep game completed", "Winner: " + userId);
          AWSXRay.endSubsegment();
        }
```

Because the request is now resolved before the call to Amazon SNS, the application creates a separate subsegment for the thread\. This prevents the X\-Ray SDK from closing the segment before it records the response from Amazon SNS\. If no subsegment is open when Scorekeep resolved the request, the response from Amazon SNS could be lost\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-workerthread.png)

See [Passing Segment Context between Threads in a Multithreaded Application](xray-sdk-java-multithreading.md) for more information about multithreading\.