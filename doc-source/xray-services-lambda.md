# AWS Lambda and AWS X\-Ray<a name="xray-services-lambda"></a>

You can use AWS X\-Ray to trace your AWS Lambda functions\. Lambda runs the [X\-Ray daemon](xray-daemon.md) and records a segment with details about the function invocation and execution\. For further instrumentation, you can bundle the X\-Ray SDK with your function to record outgoing calls and add annotations and metadata\.

If your Lambda function is called by another instrumented service, Lambda traces requests that have already been sampled without any additional configuration\. The upstream service can be an instrumented web application or another Lambda function\. Your service can invoke the function directly with an instrumented AWS SDK client, or by calling an API Gateway API with an instrumented HTTP client\.

If your Lambda function runs on a schedule, or is invoked by a service that is not instrumented, you can configure Lambda to sample and record invocations with active tracing\.

**To configure X\-Ray integration on an AWS Lambda function**

1. Open the [AWS Lambda console](https://console.aws.amazon.com/lambda)\.

1. Choose your function\.

1. Choose **Configuration**\.

1. Under **AWS X\-Ray**, enable **Active tracing**\.

On runtimes with a corresponding X\-Ray SDK, Lambda also runs the X\-Ray daemon\.

**X\-Ray SDKs on Lambda**
+ **X\-Ray SDK for Go** – Go 1\.7 and newer runtimes
+ **X\-Ray SDK for Java** – Java 8 runtime
+ **X\-Ray SDK for Node\.js** – Node\.js 4\.3 and newer runtimes
+ **X\-Ray SDK for Python** – Python 2\.7, Python 3\.6, and newer runtimes
+ **X\-Ray SDK for \.NET** – \.NET Core 2\.0 and newer runtimes

To use the X\-Ray SDK on Lambda, bundle it with your function code each time you create a new version\. You can instrument your Lambda functions with the same methods that you use to instrument applications running on other services\. The primary difference is that you don't use the SDK to instrument incoming requests, make sampling decisions, and create segments\.

The other difference between instrumenting Lambda functions and web applications is that the segment that Lambda creates and sends to X\-Ray cannot be modified by your function code\. You can create subsegments and record annotations and metadata on them, but you can't add annotations and metadata to the parent segment\.

For more information, see [Using AWS X\-Ray](https://docs.aws.amazon.com/lambda/latest/dg/lambda-x-ray.html) in the *AWS Lambda Developer Guide*\.