# AWS Distro for OpenTelemetry and AWS X\-Ray<a name="xray-services-adot"></a>

Use the AWS Distro for OpenTelemetry \(ADOT\) to collect and send metrics and traces to AWS X\-Ray and other monitoring solutions, such as Amazon CloudWatch, Amazon OpenSearch Service, and Amazon Managed Service for Prometheus\. 

## AWS Distro for OpenTelemetry<a name="xray-services-adot-intro"></a>

The AWS Distro for OpenTelemetry \(ADOT\) is an AWS distribution based on the Cloud Native Computing Foundation \(CNCF\) OpenTelemetry project\. OpenTelemetry provides a single set of open source APIs, libraries, and agents to collect distributed traces and metrics\. This toolkit is a distribution of upstream OpenTelemetry components including SDKs, auto\-instrumentation agents, and collectors that are tested, optimized, secured, and supported by AWS\. 

With ADOT, engineers can instrument their applications once and send correlated metrics and traces to multiple AWS monitoring solutions including Amazon CloudWatch, AWS X\-Ray, Amazon OpenSearch Service, and Amazon Managed Service for Prometheus\.

ADOT is integrated with a growing number of AWS services to simplify sending traces and metrics to monitoring solutions such as X\-Ray\. Some examples of services integrated with ADOT include: 
+ *AWS Lambda* – AWS managed Lambda layers for ADOT provides a plug\-and\-play user experience by automatically instrumenting a Lambda function, packaging OpenTelemetry together with an out\-of\-the\-box configuration for AWS Lambda and X\-Ray in an easy to setup layer\. Users can enable and disable OpenTelemetry for their Lambda function without changing code\. For more information, see [AWS Distro for OpenTelemetry Lambda](https://aws-otel.github.io/docs/getting-started/lambda) 
+ *Amazon Elastic Container Service \(ECS\)* – Collect metrics and traces from Amazon ECS applications using the AWS Distro for OpenTelemetry Collector, to send to X\-Ray and other monitoring solutions\. For more information, see [Collecting application trace data](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/trace-data.html) in the Amazon ECS developer guide\. 
+ *AWS App Runner* – App Runner supports sending traces to X\-Ray using the AWS Distro for OpenTelemetry \(ADOT\)\. Use ADOT SDKs to collect trace data for your containerized applications, and use X\-Ray to analyze and gain insights into your instrumented application\. For more information, see [AWS App Runner and X\-Ray](xray-services-app-runner.md)\. 

For more information about the AWS Distro for OpenTelemetry, including integration with additional AWS services, see the [AWS Distro for OpenTelemetry Documentation](https://aws-otel.github.io/docs/introduction)\. 

For more information about instrumenting your application with AWS Distro for OpenTelemetry and X\-Ray, see [Instrumenting your application with the AWS Distro for OpenTelemetry](xray-instrumenting-your-app.md#xray-instrumenting-opentel)\. 