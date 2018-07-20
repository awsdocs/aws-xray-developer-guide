# Amazon API Gateway and AWS X\-Ray<a name="xray-services-apigateway"></a>

Amazon API Gateway provides [request tracing](xray-usage.md#xray-usage-services) support for AWS X\-Ray\. An API Gateway gateway adds a [tracing header](xray-concepts.md#xray-concepts-tracingheader) to incoming HTTP requests that don't already have one\.

```
X-Amzn-Trace-Id: Root=1-5759e988-bd862e3fe1be46a994272793
```

**Trace ID Format**

A `trace_id` consists of three numbers separated by hyphens\. For example, `1-58406520-a006649127e371903a2de979`\. This includes:
+ The version number, that is, `1`\.
+ The time of the original request, in Unix epoch time, in **8 hexadecimal digits**\.

  For example, 10:00AM December 1st, 2016 PST in epoch time is `1480615200` seconds, or `58406520` in hexadecimal digits\.
+ A 96\-bit identifier for the trace, globally unique, in **24 hexadecimal digits**\.

API Gateway doesn't make sampling decisions or send trace data to X\-Ray\. However, by adding the trace header, it records the time that the request reached the gateway\. By comparing the timestamp on the trace to the timestamp on the segment sent by the instrumented service behind the gateway, you can see how long the request took to reach your service after hitting the gateway\.

If your gateway is downstream of instrumented services in your application, the gateway propagates the tracing header to your backend AWS Lambda functions or HTTP service\. The upstream service and the service fronted by the gateway are connected by the trace ID, but the gateway itself doesn't appear as a node on your service map\.

If you use API Gateway to [generate a Java SDK for your API](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-generate-sdk.html), you can instrument the SDK client by adding a request handler with the client builder, in the same way that you would manually instrument an AWS SDK client\. See [Tracing AWS SDK Calls with the X\-Ray SDK for Java](xray-sdk-java-awssdkclients.md) for instructions\.