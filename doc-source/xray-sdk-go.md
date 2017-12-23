# AWS X\-Ray SDK for Go<a name="xray-sdk-go"></a>

The X\-Ray SDK for Go is a set of libraries for Go applications that provide classes and methods for generating and sending trace data to the X\-Ray daemon\. Trace data includes information about incoming HTTP requests served by the application, and calls that the application makes to downstream services using the AWS SDK, HTTP clients, or an SQL database connector\. You can also create segments manually and add debug information in annotations and metadata\.

**Annotations and Metadata**  
Annotations and metadata are arbitrary text that you add to segments with the X\-Ray SDK\. Annotations are indexed for use with filter expressions\. Metadata are not indexed, but can be viewed in the raw segment with the X\-Ray console or API\. Anyone that you grant read access to X\-Ray can view this data\.

Download the SDK from its [GitHub repository](https://github.com/aws/aws-xray-sdk-go) with `go get`:

```
$ go get -u github.com/aws/aws-xray-sdk-go
```

Once you have downloaded the SDK, view the API documentation by:

1. Navigating to the `$GOPATH/src/github.com/aws/aws-xray-sdk-go` \(Linux or Mac\) directory or the `%GOPATH%\src\github.com\aws\aws-xray-sdk-go` \(Windows\) folder

1. Entering the following command:

   ```
   $ godoc -http=:6060
   ```

1. Opening a browser at `http://localhost:6060/pkg/github.com/aws/aws-xray-sdk-go/`\.

Start by using the `xray.Handler` function to trace incoming requests\. The message handler creates a segment for each traced request, and completes the segment when the response is sent\. While the segment is open you can use the SDK client's methods to add information to the segment and create subsegments to trace downstream calls\. The SDK also automatically records exceptions that your application throws while the segment is open\.

Next, wrap your client with a call to the `AWS` function\. This step ensures that X\-Ray instruments calls to any client methods\. You can also instrument calls to SQL databases\.

You must have the X\-Ray daemon running to buffer trace segments and forward them to the service backend\. You can configure the X\-Ray SDK for Go to communicate with the X\-Ray daemon through environment variables, by calling `Configure`, or by assuming default values\.

When you have a lot of instrumented clients in your code, a single request segment can contain a large number of subsegments, one for each call made with an instrumented client\. You can organize and group subsegments by wrapping client calls in custom subsegments\. You can create a custom subsegment for an entire function or any section of code, and record metadata and annotations on the subsegment instead of writing everything on the parent segment\.

## Requirements<a name="xray-sdk-go-requirements"></a>

The X\-Ray SDK for Go requires Go 1\.7 or later\.

The SDK depends on the following libraries at compile and runtime:

+ AWS SDK for Go version 1\.10\.0 or newer

These dependencies are declared in the SDK's `README.md` file\.