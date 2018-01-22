# AWS Lambda and AWS X\-Ray<a name="xray-services-lambda"></a>

AWS Lambda provides basic instrumentation support for all runtimes\. When you enable X\-Ray integration on your function, Lambda makes sampling decisions on incoming requests and uploads segments to X\-Ray\.

**Note**  
If your Lambda funtion is called by another instrumented service, Lambda will trace requests that have already been sampled, even if you don't enable active tracing\.

**To configure X\-Ray integration on an AWS Lambda function**

1. Open the [AWS Lambda console](https://console.aws.amazon.com/lambda)\.

1. Choose your function\.

1. Choose **Configuration**\.

1. Under **Advanced settings**, choose **Enable active tracing**\.

On runtimes with a corresponding X\-Ray SDK, Lambda also runs the X\-Ray daemon\.

**X\-Ray SDKs on Lambda**

+ **X\-Ray SDK for Go** Go 1\.7 and newer runtime

+ **X\-Ray SDK for Java** Java 8 runtime

+ **X\-Ray SDK for Node\.js** Node\.js 4\.3 and newer runtimes

+ **X\-Ray SDK for Python** Python 2\.7, Python 3\.6, and newer runtimes

You can instrument your Lambda functions with the same methods that you use to instrument applications running on other services\. The primary difference is that you do not need to use a servlet filter or middleware to create segments and make sampling decisions\.

For more information, see [Troubleshooting Lambda\-based Applications](http://docs.aws.amazon.com/lambda/latest/dg/lambda-x-ray.html) in the AWS Lambda Developer Guide\.