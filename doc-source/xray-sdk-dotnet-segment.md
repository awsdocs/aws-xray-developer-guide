# Add annotations and metadata to segments with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-segment"></a>

You can record additional information about requests, the environment, or your application with annotations and metadata\. You can add annotations and metadata to the segments that the X\-Ray SDK creates, or to custom subsegments that you create\.

**Annotations** are key\-value pairs with string, number, or Boolean values\. Annotations are indexed for use with [filter expressions](xray-console-filters.md)\. Use annotations to record data that you want to use to group traces in the console, or when calling the [https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html](https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html) API\.

**Metadata** are key\-value pairs that can have values of any type, including objects and lists, but are not indexed for use with filter expressions\. Use metadata to record additional data that you want stored in the trace but don't need to use with search\.

**Topics**
+ [Recording annotations with the X\-Ray SDK for \.NET](#xray-sdk-dotnet-segment-annotations)
+ [Recording metadata with the X\-Ray SDK for \.NET](#xray-sdk-dotnet-segment-metadata)

## Recording annotations with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-segment-annotations"></a>

Use annotations to record information on segments or subsegments that you want indexed for search\.

**Annotation Requirements**
+ **Keys** – Up to 500 alphanumeric characters\. No spaces or symbols except underscores\.
+ **Values** – Up to 1,000 Unicode characters\.
+ **Entries** – Up to 50 annotations per trace\.

**To record annotations**

1. Get an instance of `AWSXRayRecorder`\.

   ```
   using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
   ...
   AWSXRayRecorder recorder = AWSXRayRecorder.Instance;
   ```

1. Call `addAnnotation` with a String key and a Boolean, Int32, Int64, Double, or String value\.

   ```
   recorder.AddAnnotation("mykey", "my value");
   ```

The SDK records annotations as key\-value pairs in an `annotations` object in the segment document\. Calling `addAnnotation` twice with the same key overwrites previously recorded values on the same segment or subsegment\.

To find traces that have annotations with specific values, use the `annotations.key` keyword in a [filter expression](xray-console-filters.md)\.

## Recording metadata with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-segment-metadata"></a>

Use metadata to record information on segments or subsegments that you don't need indexed for search\. Metadata values can be Strings, Numbers, Booleans, or any other Object that can be serialized into a JSON object or array\.

**To record metadata**

1. Get an instance of `AWSXRayRecorder`\.

   ```
   using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
   ...
   AWSXRayRecorder recorder = AWSXRayRecorder.Instance;
   ```

1. Call `AddMetadata` with a String namespace, String key, and an Object value\.

   ```
   segment.AddMetadata("my namespace", "my key", "my value");
   ```

   or

   Call `putMetadata` with just a key and value\.

   ```
   segment.AddMetadata("my key", "my value");
   ```

If you don't specify a namespace, the SDK uses `default`\. Calling `AddMetadata` twice with the same key overwrites previously recorded values on the same segment or subsegment\.