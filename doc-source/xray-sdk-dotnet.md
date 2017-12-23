# AWS X\-Ray SDK for \.NET<a name="xray-sdk-dotnet"></a>

The X\-Ray SDK for \.NET is a library for C\# \.NET web applications that provides classes and methods for generating and sending trace data to the X\-Ray daemon\. Trace data includes information about incoming HTTP requests served by the application, and calls that the application makes to downstream AWS services, HTTP web APIs, and SQL databases\. You can also create segments manually and add debug information in annotations and metadata\.

Download the X\-Ray SDK for \.NET from NuGet: [nuget\.org/packages/AWSXRayRecorder/](https://www.nuget.org/packages/AWSXRayRecorder/)

Start by adding a `TracingMessageHandler` to your web configuration to trace incoming requests\. The message handler creates a segment for each traced request, and completes the segment when the response is sent\. While the segment is open you can use the SDK client's methods to add information to the segment and create subsegments to trace downstream calls\. The SDK also automatically records exceptions that your application throws while the segment is open\.

Next, use the X\-Ray SDK for \.NET to instrument your AWS SDK for \.NET clients\. Whenever you make a call to a downstream AWS service or resource with an instrumented client, the SDK records information about the call in a subsegment\. AWS services and the resources that you access within the services appear as downstream nodes on the service map to help you identify errors and throttling issues on individual connections\.

The X\-Ray SDK for \.NET also provides instrumentation for downstream calls to HTTP web APIs and SQL databases\. The `GetResponseTraced` extension method for `System.Net.HttpWebRequest` traces outgoing HTTP calls\. You can use the X\-Ray SDK for \.NET's version of `SqlCommand` to instrument SQL queries\.

Once you get going with the SDK, customize its behavior by configuring the recorder and message handler\. You can add plugins to record data about the compute resources running your application, customize sampling behavior by defining sampling rules, and set the log level to see more or less information from the SDK in your application logs\.

Record additional information about requests and the work that your application does in annotations and metadata\. Annotations are simple key\-value pairs that are indexed for use with filter expressions, so that you can search for traces that contain specific data\. Metadata entries are less restrictive and can record entire objects and arrays — anything that can be serialized into JSON\.

**Annotations and Metadata**  
Annotations and metadata are arbitrary text that you add to segments with the X\-Ray SDK\. Annotations are indexed for use with filter expressions\. Metadata are not indexed, but can be viewed in the raw segment with the X\-Ray console or API\. Anyone that you grant read access to X\-Ray can view this data\.

When you have a lot of instrumented clients in your code, a single request segment can contain a large number of subsegments, one for each call made with an instrumented client\. You can organize and group subsegments by wrapping client calls in custom subsegments\. You can create a custom subsegment for an entire function or any section of code, and record metadata and annotations on the subsegment instead of writing everything on the parent segment\.

For reference documentation about the SDK's classes and methods, see the [AWS X\-Ray SDK for \.NET API Reference](http://docs.aws.amazon.com//xray-sdk-for-dotnet/latest/reference)\.

## Requirements<a name="xray-sdk-requirements"></a>

The X\-Ray SDK for \.NET requires the \.NET framework and AWS SDK for \.NET\.

## Adding the X\-Ray SDK for \.NET to Your Application<a name="xray-sdk-dotnet-dependencies"></a>

Use NuGet to add the X\-Ray SDK for \.NET to your application\.

**To install the X\-Ray SDK for \.NET with NuGet Package Manager in Visual Studio**

1. Choose **Tools**, choose **NuGet Package Manager**, and then choose **Manage NuGet Packages for Solution**\.

1. Search for **AWSXRayRecorder**\.

1. Choose the package and then choose **Install**\.