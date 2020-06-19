# Amazon SQS and AWS X\-Ray<a name="xray-services-sqs"></a>

AWS X\-Ray integrates with Amazon Simple Queue Service \(Amazon SQS\) to trace messages that are passed through an Amazon SQS queue\. If a service traces requests by using the X\-Ray SDK, Amazon SQS can send the tracing header and continue to propagate the original trace from the sender to the consumer with a consistent trace ID\. Trace continuity enables users to track, analyze, and debug throughout downstream services\.

![\[Service map from Lambda through the Amazon SQS queue.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/sqs-manual-servicemap.png)

Amazon SQS supports the following tracing header instrumentation:
+ **Default HTTP Header** – The X\-Ray SDK automatically populates the trace header as an HTTP header when you call Amazon SQS through the AWS SDK\. The default trace header is carried by `X-Amzn-Trace-Id` and corresponds to all messages included in a [https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html) or [https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessageBatch.html](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessageBatch.html) request\. To learn more about the default HTTP header, see [Tracing header](xray-concepts.md#xray-concepts-tracingheader)\.
+ **`AWSTraceHeader` System Attribute** – The `AWSTraceHeader` is a [message system attribute](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_MessageSystemAttributeValue.html) reserved by Amazon SQS to carry the X\-Ray trace header with messages in the queue\. `AWSTraceHeader` is available for use even when auto\-instrumentation through the X\-Ray SDK is not, for example when building a tracing SDK for a new language\. When both header instrumentations are set, the message system attribute overrides the HTTP trace header\.

When running on Amazon EC2, Amazon SQS supports processing one message at a time\. This applies when running on an on\-premises host, and when using container services, such as AWS Fargate, Amazon ECS, or AWS App Mesh\. 

The trace header is excluded from both Amazon SQS message size and message attribute quoatas\. Enabling X\-Ray tracing will not exceed your Amazon SQS quotas\. To learn more about AWS quotas, see [Amazon SQS Quotas](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-limits.html)\.

## Send the HTTP trace header<a name="xray-services-sqs-sending"></a>

Sender components in Amazon SQS can send the trace header automatically through the [https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessageBatch.html](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessageBatch.html) or [https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html) call\. When AWS SDK clients are instrumented, they can be automatically tracked through all languages supported through the X\-Ray SDK\. Traced AWS services and resources that you access within those services \(for example, an Amazon S3 bucket or Amazon SQS queue\), appear as downstream nodes on the service map in the X\-Ray console\.

To learn how to trace AWS SDK calls with your preferred language, see the following topics in the supported SDKs:
+ Go – [Tracing AWS SDK calls with the X\-Ray SDK for Go](xray-sdk-go-awssdkclients.md)
+ Java – [Tracing AWS SDK calls with the X\-Ray SDK for Java](xray-sdk-java-awssdkclients.md)
+ Node\.js – [Tracing AWS SDK calls with the X\-Ray SDK for Node\.js](xray-sdk-nodejs-awssdkclients.md)
+ Python – [Tracing AWS SDK calls with the X\-Ray SDK for Python](xray-sdk-python-awssdkclients.md)
+ Ruby – [Tracing AWS SDK calls with the X\-Ray SDK for Ruby](xray-sdk-ruby-awssdkclients.md)
+ \.NET – [Tracing AWS SDK calls with the X\-Ray SDK for \.NET](xray-sdk-dotnet-sdkclients.md)

## Retrieve the trace header and recover trace context<a name="xray-services-sqs-retrieving"></a>

To continue context propagation with Amazon SQS, you must manually instrument the handoff to the receiver component\.

There are three main steps to recovering the trace context:
+ Receive the message from the queue for the `AWSTraceHeader` attribute by calling the [https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ReceiveMessage.html](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ReceiveMessage.html) API\.
+ Retrieve the trace header from the attribute\.
+ Recover the trace ID from the header\. Optionally, add more metrics to the segment\.

The following is an example implementation written with the X\-Ray SDK for Java\.

**Example : Retrieve the trace header and recover trace context**  

```
// Receive the message from the queue, specifying the "AWSTraceHeader"
ReceiveMessageRequest receiveMessageRequest = new ReceiveMessageRequest()
        .withQueueUrl(QUEUE_URL)
        .withAttributeNames("AWSTraceHeader");
List<Message> messages = sqs.receiveMessage(receiveMessageRequest).getMessages();

if (!messages.isEmpty()) {
    Message message = messages.get(0);
    
    // Retrieve the trace header from the AWSTraceHeader message system attribute
    String traceHeaderStr = message.getAttributes().get("AWSTraceHeader");
    if (traceHeaderStr != null) {
        TraceHeader traceHeader = TraceHeader.fromString(traceHeaderStr);

        // Recover the trace context from the trace header
        Segment segment = AWSXRay.getCurrentSegment();
        segment.setTraceId(traceHeader.getRootTraceId());
        segment.setParentId(traceHeader.getParentId());
        segment.setSampled(traceHeader.getSampled().equals(TraceHeader.SampleDecision.SAMPLED));
    }
}
```