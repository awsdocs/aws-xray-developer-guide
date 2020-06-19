# AWS X\-Ray SDK for Ruby<a name="xray-sdk-ruby"></a>

The X\-Ray SDK is a library for Ruby web applications that provides classes and methods for generating and sending trace data to the X\-Ray daemon\. Trace data includes information about incoming HTTP requests served by the application, and calls that the application makes to downstream services using the AWS SDK, HTTP clients, or an active record client\. You can also create segments manually and add debug information in annotations and metadata\.

You can download the SDK by adding it to your gemfile and running `bundle install`\.

**Example Gemfile**  

```
gem 'aws-sdk'
```

If you use Rails, start by [adding the X\-Ray SDK middleware](xray-sdk-ruby-middleware.md) to trace incoming requests\. A request filter creates a [segment](xray-concepts.md#xray-concepts-segments)\. While the segment is open, you can use the SDK client's methods to add information to the segment and create subsegments to trace downstream calls\. The SDK also automatically records exceptions that your application throws while the segment is open\. For non\-Rails applications, you can [create segments manually](xray-sdk-ruby-middleware.md#xray-sdk-ruby-middleware-manual)\.

Next, use the X\-Ray SDK to instrument your AWS SDK for Ruby, HTTP, and SQL clients by [configuring the recorder](xray-sdk-ruby-patching.md) to patch the associated libraries\. Whenever you make a call to a downstream AWS service or resource with an instrumented client, the SDK records information about the call in a subsegment\. AWS services and the resources that you access within the services appear as downstream nodes on the service map to help you identify errors and throttling issues on individual connections\.

Once you get going with the SDK, customize its behavior by [configuring the recorder](xray-sdk-ruby-configuration.md)\. You can add plugins to record data about the compute resources running your application, customize sampling behavior by defining sampling rules, and provide a logger to see more or less information from the SDK in your application logs\.

Record additional information about requests and the work that your application does in [annotations and metadata](xray-sdk-ruby-segment.md)\. Annotations are simple key\-value pairs that are indexed for use with [filter expressions](xray-console-filters.md), so that you can search for traces that contain specific data\. Metadata entries are less restrictive and can record entire objects and arrays — anything that can be serialized into JSON\.

**Annotations and Metadata**  
Annotations and metadata are arbitrary text that you add to segments with the X\-Ray SDK\. Annotations are indexed for use with filter expressions\. Metadata are not indexed, but can be viewed in the raw segment with the X\-Ray console or API\. Anyone that you grant read access to X\-Ray can view this data\.

When you have a lot of instrumented clients in your code, a single request segment can contain a large number of subsegments, one for each call made with an instrumented client\. You can organize and group subsegments by wrapping client calls in [custom subsegments](xray-sdk-ruby-subsegments.md)\. You can create a custom subsegment for an entire function or any section of code, and record metadata and annotations on the subsegment instead of writing everything on the parent segment\.

For reference documentation for the SDK's classes and methods, see the [AWS X\-Ray SDK for Ruby API Reference](https://docs.aws.amazon.com/xray-sdk-for-ruby/latest/reference)\.

## Requirements<a name="xray-sdk-ruby-requirements"></a>

The X\-Ray SDK requires Ruby 2\.3 or later and is compatible with the following libraries:
+ AWS SDK for Ruby version 3\.0 or later
+ Rails version 5\.1 or later