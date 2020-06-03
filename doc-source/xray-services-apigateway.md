# Amazon API Gateway active tracing support for AWS X\-Ray<a name="xray-services-apigateway"></a>

You can use X\-Ray to trace and analyze user requests as they travel through your Amazon API Gateway APIs to the underlying services\. API Gateway supports X\-Ray tracing for all API Gateway endpoint types: Regional, edge\-optimized, and private\. You can use X\-Ray with Amazon API Gateway in all AWS Regions where X\-Ray is available\. For more information, see [Trace API Gateway API Execution with AWS X\-Ray](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-xray.html) in the Amazon API Gateway Developer Guide\.

Amazon API Gateway provides [active tracing](xray-usage.md#xray-usage-services) support for AWS X\-Ray\. Enable active tracing on your API stages to sample incoming requests and send traces to X\-Ray\.

**To enable active tracing on an API stage**

1. Open the API Gateway console at [https://console\.aws\.amazon\.com/apigateway/](https://console.aws.amazon.com/apigateway/)\. 

1. Choose an API\.

1. Choose a stage\.

1. On the **Logs/Tracing** tab, choose **Enable X\-Ray Tracing**\.

1. Choose **Resources** in the left side navigation panel\.

1. To redeploy the API with the new settings, choose **Actions**, **Deploy API**\.

API Gateway uses sampling rules that you define in the X\-Ray console to determine which requests to record\. You can create rules that only apply to APIs, or that apply only to requests that contain certain headers\. API Gateway records headers in attributes on the segment, along with details about the stage and request\. For more information, see [Configuring sampling rules in the X\-Ray console](xray-console-sampling.md)\.

For all incoming requests, API Gateway adds a [tracing header](xray-concepts.md#xray-concepts-tracingheader) to incoming HTTP requests that don't already have one\.

```
X-Amzn-Trace-Id: Root=1-5759e988-bd862e3fe1be46a994272793
```

**Trace ID Format**

A `trace_id` consists of three numbers separated by hyphens\. For example, `1-58406520-a006649127e371903a2de979`\. This includes:
+ The version number, that is, `1`\.
+ The time of the original request, in Unix epoch time, in **8 hexadecimal digits**\.

  For example, 10:00AM December 1st, 2016 PST in epoch time is `1480615200` seconds, or `58406520` in hexadecimal digits\.
+ A 96\-bit identifier for the trace, globally unique, in **24 hexadecimal digits**\.

If active tracing is disabled, the stage still records a segment if the request comes from a service that sampled the request and started a trace\. For example, an instrumented web application can call an API Gateway API with an HTTP client\. When you instrument an HTTP client with the X\-Ray SDK, it adds a tracing header to the outgoing request that contains the sampling decision\. API Gateway reads the tracing header and creates a segment for sampled requests\.

If you use API Gateway to [generate a Java SDK for your API](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-generate-sdk.html), you can instrument the SDK client by adding a request handler with the client builder, in the same way that you would manually instrument an AWS SDK client\. See [Tracing AWS SDK calls with the X\-Ray SDK for Java](xray-sdk-java-awssdkclients.md) for instructions\.