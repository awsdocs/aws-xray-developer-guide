# AWS X\-Ray SDK for Go<a name="xray-sdk-go"></a>

The X\-Ray SDK for Go is a set of libraries for Go applications that provide classes and methods for generating and sending trace data to the X\-Ray daemon\. Trace data includes information about incoming HTTP requests served by the application, and calls that the application makes to downstream services using the AWS SDK, HTTP clients, or an SQL database connector\. You can also create segments manually and add debug information in annotations and metadata\.

Download the SDK from its [GitHub repository](https://github.com/aws/aws-xray-sdk-go) with `go get`:

```
$ go get -u github.com/aws/aws-xray-sdk-go/... 
```

For web applications, start by [using the `xray.Handler` function](xray-sdk-go-handler.md) to trace incoming requests\. The message handler creates a [segment](xray-concepts.md#xray-concepts-segments) for each traced request, and completes the segment when the response is sent\. While the segment is open you can use the SDK client's methods to add information to the segment and create subsegments to trace downstream calls\. The SDK also automatically records exceptions that your application throws while the segment is open\.

For Lambda functions called by an instrumented application or service, Lambda reads the [tracing header](xray-concepts.md#xray-concepts-tracingheader) and traces sampled requests automatically\. For other functions, you can [configure Lambda](xray-services-lambda.md) to sample and trace incoming requests\. In either case, Lambda creates the segment and provides it to the X\-Ray SDK\.

**Note**  
On Lambda, the X\-Ray SDK is optional\. If you don't use it in your function, your service map will still include a node for the Lambda service, and one for each Lambda function\. By adding the SDK, you can instrument your function code to add subsegments to the function segment recorded by Lambda\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

Next, [wrap your client with a call to the `AWS` function](xray-sdk-go-awssdkclients.md)\. This step ensures that X\-Ray instruments calls to any client methods\. You can also [instrument calls to SQL databases](xray-sdk-go-sqlclients.md)\.

Once you get going with the SDK, customize its behavior by [configuring the recorder and middleware](xray-sdk-go-configuration.md)\. You can add plugins to record data about the compute resources running your application, customize sampling behavior by defining sampling rules, and set the log level to see more or less information from the SDK in your application logs\.

Record additional information about requests and the work that your application does in [annotations and metadata](xray-sdk-go-segment.md)\. Annotations are simple key\-value pairs that are indexed for use with [filter expressions](xray-console-filters.md), so that you can search for traces that contain specific data\. Metadata entries are less restrictive and can record entire objects and arrays — anything that can be serialized into JSON\.

**Annotations and Metadata**  
Annotations and metadata are arbitrary text that you add to segments with the X\-Ray SDK\. Annotations are indexed for use with filter expressions\. Metadata are not indexed, but can be viewed in the raw segment with the X\-Ray console or API\. Anyone that you grant read access to X\-Ray can view this data\.

When you have a lot of instrumented clients in your code, a single request segment can contain a large number of subsegments, one for each call made with an instrumented client\. You can organize and group subsegments by wrapping client calls in [custom subsegments](xray-sdk-go-subsegments.md)\. You can create a custom subsegment for an entire function or any section of code, and record metadata and annotations on the subsegment instead of writing everything on the parent segment\.

## Requirements<a name="xray-sdk-go-requirements"></a>

The X\-Ray SDK for Go requires Go 1\.9 or later\.

The SDK depends on the following libraries at compile and runtime:
+ AWS SDK for Go version 1\.10\.0 or newer

These dependencies are declared in the SDK's `README.md` file\.

## Reference documentation<a name="xray-sdk-go-reference"></a>

Once you have downloaded the SDK, build and host the documentation locally to view it in a web browser\.

**To view the reference documentation**

1. Navigating to the `$GOPATH/src/github.com/aws/aws-xray-sdk-go` \(Linux or Mac\) directory or the `%GOPATH%\src\github.com\aws\aws-xray-sdk-go` \(Windows\) folder

1. Run the `godoc` command\.

   ```
   $ godoc -http=:6060
   ```

1. Opening a browser at `http://localhost:6060/pkg/github.com/aws/aws-xray-sdk-go/`\.