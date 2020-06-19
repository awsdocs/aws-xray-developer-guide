# AWS X\-Ray SDK for \.NET<a name="xray-sdk-dotnet"></a>

The X\-Ray SDK for \.NET is a library for instrumenting C\# \.NET web applications, \.NET Core web applications, and \.NET Core functions on AWS Lambda\. It provides classes and methods for generating and sending trace data to the [X\-Ray daemon](xray-daemon.md)\. This includes information about incoming requests served by the application, and calls that the application makes to downstream AWS services, HTTP web APIs, and SQL databases\.

**Note**  
The X\-Ray SDK for \.NET is an open source project\. You can follow the project and submit issues and pull requests on GitHub: [github\.com/aws/aws\-xray\-sdk\-dotnet](https://github.com/aws/aws-xray-sdk-dotnet)

For web applications, start by [adding a message handler to your web configuration](xray-sdk-dotnet-messagehandler.md) to trace incoming requests\. The message handler creates a [segment](xray-concepts.md#xray-concepts-segments) for each traced request, and completes the segment when the response is sent\. While the segment is open you can use the SDK client's methods to add information to the segment and create subsegments to trace downstream calls\. The SDK also automatically records exceptions that your application throws while the segment is open\.

For Lambda functions called by an instrumented application or service, Lambda reads the [tracing header](xray-concepts.md#xray-concepts-tracingheader) and traces sampled requests automatically\. For other functions, you can [configure Lambda](xray-services-lambda.md) to sample and trace incoming requests\. In either case, Lambda creates the segment and provides it to the X\-Ray SDK\.

**Note**  
On Lambda, the X\-Ray SDK is optional\. If you don't use it in your function, your service map will still include a node for the Lambda service, and one for each Lambda function\. By adding the SDK, you can instrument your function code to add subsegments to the function segment recorded by Lambda\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

Next, use the X\-Ray SDK for \.NET to [instrument your AWS SDK for \.NET clients](xray-sdk-dotnet-sdkclients.md)\. Whenever you make a call to a downstream AWS service or resource with an instrumented client, the SDK records information about the call in a subsegment\. AWS services and the resources that you access within the services appear as downstream nodes on the service map to help you identify errors and throttling issues on individual connections\.

The X\-Ray SDK for \.NET also provides instrumentation for downstream calls to [HTTP web APIs](xray-sdk-dotnet-httpclients.md) and [SQL databases](xray-sdk-dotnet-sqlqueries.md)\. The `GetResponseTraced` extension method for `System.Net.HttpWebRequest` traces outgoing HTTP calls\. You can use the X\-Ray SDK for \.NET's version of `SqlCommand` to instrument SQL queries\.

Once you get going with the SDK, customize its behavior by [configuring the recorder and message handler](xray-sdk-dotnet-configuration.md)\. You can add plugins to record data about the compute resources running your application, customize sampling behavior by defining sampling rules, and set the log level to see more or less information from the SDK in your application logs\.

Record additional information about requests and the work that your application does in [annotations and metadata](xray-sdk-dotnet-segment.md)\. Annotations are simple key\-value pairs that are indexed for use with [filter expressions](xray-console-filters.md), so that you can search for traces that contain specific data\. Metadata entries are less restrictive and can record entire objects and arrays — anything that can be serialized into JSON\.

**Annotations and Metadata**  
Annotations and metadata are arbitrary text that you add to segments with the X\-Ray SDK\. Annotations are indexed for use with filter expressions\. Metadata are not indexed, but can be viewed in the raw segment with the X\-Ray console or API\. Anyone that you grant read access to X\-Ray can view this data\.

When you have many instrumented clients in your code, a single request segment can contain a large number of subsegments, one for each call made with an instrumented client\. You can organize and group subsegments by wrapping client calls in [custom subsegments](xray-sdk-dotnet-subsegments.md)\. You can create a custom subsegment for an entire function or any section of code, and record metadata and annotations on the subsegment instead of writing everything on the parent segment\.

For reference documentation about the SDK's classes and methods, see the following:
+ [AWS X\-Ray SDK for \.NET API Reference](https://docs.aws.amazon.com//xray-sdk-for-dotnet/latest/reference)
+ [AWS X\-Ray SDK for \.NET Core API Reference](https://docs.aws.amazon.com//xray-sdk-for-dotnetcore/latest/reference)

The same package supports both \.NET and \.NET Core, but the classes that are used vary\. Examples in this chapter link to the \.NET API reference unless the class is specific to \.NET Core\.

## Requirements<a name="xray-sdk-requirements"></a>

The X\-Ray SDK for \.NET requires the \.NET Framework 4\.5 or later and AWS SDK for \.NET\.

For \.NET Core applications and functions, the SDK requires \.NET Core 2\.0 or later\.

## Adding the X\-Ray SDK for \.NET to your application<a name="xray-sdk-dotnet-dependencies"></a>

Use NuGet to add the X\-Ray SDK for \.NET to your application\.

**To install the X\-Ray SDK for \.NET with NuGet package manager in Visual Studio**

1. Choose **Tools**, **NuGet Package Manager**, **Manage NuGet Packages for Solution**\.

1. Search for **AWSXRayRecorder**\.

1. Choose the package, and then choose **Install**\.