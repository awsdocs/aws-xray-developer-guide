# Working with Go<a name="xray-go"></a>

 There are two ways to instrument your Go application to send traces to X\-Ray: 
+ [AWS Distro for OpenTelemetry Go](xray-go-opentel-sdk.md) – An AWS distribution that provides a set of open source libraries for sending correlated metrics and traces to multiple AWS monitoring solutions including Amazon CloudWatch, AWS X\-Ray, and Amazon OpenSearch Service, via the [AWS Distro for OpenTelemetry Collector](https://aws-otel.github.io/docs/getting-started/collector)\.
+ [AWS X\-Ray SDK for Go](xray-sdk-go.md) – A set of libraries for generating and sending traces to X\-Ray via the [X\-Ray daemon](xray-daemon.md)\.

 For more information, see [Choosing between the AWS Distro for OpenTelemetry and X\-Ray SDKs](xray-instrumenting-your-app.md#xray-instrumenting-choosing)\. 