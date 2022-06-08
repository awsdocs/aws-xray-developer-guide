# Instrumenting your application for AWS X\-Ray<a name="xray-instrumenting-your-app"></a>

Instrumenting your application involves sending trace data for incoming and outbound requests and other events within your application, along with metadata about each request\. There are several different instrumentation options you can choose from or combine, based on your particular requirements: 
+ *Auto instrumentation* – instrument your application with zero code changes, typically via configuration changes, adding an auto\-instrumentation agent, or other mechanisms\. 
+ *Library instrumentation* – make minimal application code changes to add pre\-built instrumentation targeting specific libraries or frameworks, such as the AWS SDK, Apache HTTP clients, or SQL clients\. 
+ *Manual instrumentation* – add instrumentation code to your application at each location where you want to send trace information\. 

 There are several SDKs, agents, and tools that can be used to instrument your application for X\-Ray tracing\. 

**Topics**
+ [Instrumenting your application with the AWS Distro for OpenTelemetry](#xray-instrumenting-opentel)
+ [Instrumenting your application with AWS X\-Ray SDKs](#xray-instrumenting-xray-sdk)
+ [Choosing between the AWS Distro for OpenTelemetry and X\-Ray SDKs](#xray-instrumenting-choosing)

## Instrumenting your application with the AWS Distro for OpenTelemetry<a name="xray-instrumenting-opentel"></a>

The AWS Distro for OpenTelemetry \(ADOT\) is an AWS distribution based on the Cloud Native Computing Foundation \(CNCF\) OpenTelemetry project\. OpenTelemetry provides a single set of open source APIs, libraries, and agents to collect distributed traces and metrics\. This toolkit is a distribution of upstream OpenTelemetry components including SDKs, auto\-instrumentation agents, and collectors that are tested, optimized, secured, and supported by AWS\. 

With ADOT, engineers can instrument their applications once and send correlated metrics and traces to multiple AWS monitoring solutions including Amazon CloudWatch, AWS X\-Ray, and Amazon OpenSearch Service\.

Using X\-Ray with ADOT requires two components: an *OpenTelemetry SDK* enabled for use with X\-Ray, and the *AWS Distro for OpenTelemetry Collector* enabled for use with X\-Ray\. For more information about using the AWS Distro for OpenTelemetry with AWS X\-Ray and other AWS services, see the [AWS Distro for OpenTelemetry Documentation](https://aws-otel.github.io/docs/introduction)\.

For more information about language support and usage, see [AWS Observability on GitHub](https://github.com/aws-observability)\.

ADOT includes the following:
+ [AWS Distro for OpenTelemetry Go](https://aws-otel.github.io/docs/getting-started/go-sdk)
+ [AWS Distro for OpenTelemetry Java](https://aws-otel.github.io/docs/getting-started/java-sdk)
+ [AWS Distro for OpenTelemetry JavaScript](https://aws-otel.github.io/docs/getting-started/javascript-sdk)
+ [AWS Distro for OpenTelemetry Python](https://aws-otel.github.io/docs/getting-started/python-sdk)
+ [AWS Distro for OpenTelemetry \.NET](https://aws-otel.github.io/docs/getting-started/dotnet-sdk)

ADOT currently includes auto\-instrumentation support for [Java](https://aws-otel.github.io/docs/getting-started/java-sdk/trace-auto-instr) and [Python](https://aws-otel.github.io/docs/getting-started/python-sdk/trace-auto-instr)\. In addition, ADOT enables auto\-instrumentation of AWS Lambda functions and their downstream requests using Java, Node\.js, and Python runtimes, via [ADOT Managed Lambda Layers](https://aws-otel.github.io/docs/getting-started/lambda)\. 

## Instrumenting your application with AWS X\-Ray SDKs<a name="xray-instrumenting-xray-sdk"></a>

 AWS X\-Ray includes a set of language\-specific SDKs for instrumenting your application to send traces to X\-Ray\. Each X\-Ray SDK provides the following: 
+ *Interceptors* to add to your code to trace incoming HTTP requests
+ *Client handlers* to instrument AWS SDK clients that your application uses to call other AWS services
+ An *HTTP client* to instrument calls to other internal and external HTTP web services

X\-Ray SDKs also support instrumenting calls to SQL databases, automatic AWS SDK client instrumentation, and other features\. Instead of sending trace data directly to X\-Ray, the SDK sends JSON segment documents to a daemon process listening for UDP traffic\. The [X\-Ray daemon](xray-daemon.md) buffers segments in a queue and uploads them to X\-Ray in batches\. 

The following language\-specific SDKs are provided:
+ [AWS X\-Ray SDK for Go](xray-sdk-go.md)
+ [AWS X\-Ray SDK for Java](xray-sdk-java.md)
+ [AWS X\-Ray SDK for Node\.js](xray-sdk-nodejs.md)
+ [AWS X\-Ray SDK for Python](xray-sdk-python.md)
+ [AWS X\-Ray SDK for \.NET](xray-sdk-dotnet.md)
+ [AWS X\-Ray SDK for Ruby](xray-sdk-ruby.md)

X\-Ray currently includes auto\-instrumentation support for [Java](aws-x-ray-auto-instrumentation-agent-for-java.md)\. 

## Choosing between the AWS Distro for OpenTelemetry and X\-Ray SDKs<a name="xray-instrumenting-choosing"></a>

 The SDKs included with X\-Ray are part of a tightly integrated instrumentation solution offered by AWS\. The AWS Distro for OpenTelemetry is part of a broader industry solution in which X\-Ray is only one of many tracing solutions\. You can implement end\-to\-end tracing in X\-Ray using either approach, but it’s important to understand the differences in order to determine the most useful approach for you\. 

 We recommend instrumenting your application with the AWS Distro for OpenTelemetry if you need the following: 
+ The ability to send traces to multiple different tracing backends without having to re\-instrument your code
+ Support for a large number of library instrumentations for each language, maintained by the OpenTelemetry community

 We recommend choosing an X\-Ray SDK for instrumenting your application if you need the following: 
+ A tightly integrated single\-vendor solution
+ Integration with X\-Ray centralized sampling rules, including the ability to configure sampling rules from the X\-Ray console and automatically use them across multiple hosts