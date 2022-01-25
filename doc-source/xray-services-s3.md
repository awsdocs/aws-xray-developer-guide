# Amazon S3 and AWS X\-Ray<a name="xray-services-s3"></a>

AWS X\-Ray integrates with Amazon S3 to trace upstream requests to update your application's S3 buckets\. If a service traces requests by using the X\-Ray SDK, Amazon S3 can send the tracing headers to downstream event subscribers such as AWS Lambda, Amazon SQS, and Amazon SNS\. X\-Ray enables trace messages for Amazon S3 event notifications\.

You can use the X\-Ray service map to view the connections between Amazon S3 and other services that your application uses\. You can also use the console to view metrics such as average latency and failure rates\. For more information about the X\-Ray console, see [AWS X\-Ray console](xray-console.md)\.

Amazon S3 supports the *default http header* instrumentation\. The X\-Ray SDK automatically populates the trace header as an HTTP header when you call Amazon S3 through the AWS SDK\. The default trace header is carried by `X-Amzn-Trace-Id`\. To learn more about tracing headers, see [Tracing header](xray-concepts.md#xray-concepts-tracingheader) on the concept page\. Amazon S3 trace context propagation supports the following subscribers: Lambda, SQS, and SNS\. Because SQS and SNS are do not emit segment data themselves, they may not appear in your trace or service map when triggered by S3, even though they will propagate the trace context.

## Configure Amazon S3 event notifications<a name="xray-services-s3-notification"></a>

With the Amazon S3 notification feature, you receive notifications when certain events happen in your bucket\. These notifications can then be propagated to the following destinations within your application:
+ Amazon Simple Notification Service \(Amazon SNS\)
+ Amazon Simple Queue Service \(Amazon SQS\)
+ AWS Lambda

For a list of supported events, see [Supported event types in the Amazon S3 developer guide](https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html#supported-notification-event-types)\.

### Amazon SNS and Amazon SQS<a name="xray-services-s3-notifications-snssqs"></a>

To publish notifications to an SNS topic or an SQS queue, you must first grant Amazon S3 permissions\. To grant these permissions, you attach an AWS Identity and Access Management \(IAM\) policy to the destination SNS topic or SQS queue\. To learn more about the IAM policies required, see [Granting permissions to publish messages to an SNS topic or an SQS queue](https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html#grant-sns-sqs-permission-for-s3)\. 

For information about integrating SNS and SQS with X\-Ray see, [Amazon SNS and AWS X\-Ray](xray-services-sns.md) and [Amazon SQS and AWS X\-Ray](xray-services-sqs.md)\.

### AWS Lambda<a name="xray-services-s3-notifications-lambda"></a>

When you use the Amazon S3 console to configure event notifications on an S3 bucket for a Lambda function, the console sets up the necessary permissions on the Lambda function so that Amazon S3 has permissions to invoke the function from the bucket\. For more information, see [How Do I Enable and Configure Event Notifications for an S3 Bucket?](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/enable-event-notifications.html) in the Amazon Simple Storage Service Console User Guide\.

You can also grant Amazon S3 permissions from AWS Lambda to invoke your Lambda function\. For more information, see [Tutorial: Using AWS Lambda with Amazon S3](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html) in the AWS Lambda Developer Guide\. 

For more information about integrating Lambda with X\-Ray, see [Instrumenting Java code in AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/java-tracing.html)\.
