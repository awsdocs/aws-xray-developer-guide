# Add annotations and metadata to segments with the X\-Ray SDK for Go<a name="xray-sdk-go-segment"></a>

You can record additional information about requests, the environment, or your application with annotations and metadata\. You can add annotations and metadata to the segments that the X\-Ray SDK creates, or to custom subsegments that you create\.

**Annotations** are key\-value pairs with string, number, or Boolean values\. Annotations are indexed for use with [filter expressions](xray-console-filters.md)\. Use annotations to record data that you want to use to group traces in the console, or when calling the [https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html](https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html) API\.

**Metadata** are key\-value pairs that can have values of any type, including objects and lists, but are not indexed for use with filter expressions\. Use metadata to record additional data that you want stored in the trace but don't need to use with search\.

In addition to annotations and metadata, you can also [record user ID strings](#xray-sdk-go-segment-userid) on segments\. User IDs are recorded in a separate field on segments and are indexed for use with search\.

**Topics**
+ [Recording annotations with the X\-Ray SDK for Go](#xray-sdk-go-segment-annotations)
+ [Recording metadata with the X\-Ray SDK for Go](#xray-sdk-go-segment-metadata)
+ [Recording user IDs with the X\-Ray SDK for Go](#xray-sdk-go-segment-userid)

## Recording annotations with the X\-Ray SDK for Go<a name="xray-sdk-go-segment-annotations"></a>

Use annotations to record information on segments that you want indexed for search\.

**Annotation Requirements**
+ **Keys** – Up to 500 alphanumeric characters\. No spaces or symbols except underscores\.
+ **Values** – Up to 1,000 Unicode characters\.
+ **Entries** – Up to 50 annotations per trace\.

To record annotations, call `AddAnnotation` with a string containing the metadata you want to associate with the segment\.

```
xray.AddAnnotation(key string, value interface{})
```

The SDK records annotations as key\-value pairs in an `annotations` object in the segment document\. Calling `AddAnnotation` twice with the same key overwrites previously recorded values on the same segment\.

To find traces that have annotations with specific values, use the `annotations.key` keyword in a [filter expression](xray-console-filters.md)\.

## Recording metadata with the X\-Ray SDK for Go<a name="xray-sdk-go-segment-metadata"></a>

Use metadata to record information on segments that you don't need indexed for search\.

To record metadata, call `AddMetadata` with a string containing the metadata you want to associate with the segment\.

```
xray.AddMetadata(key string, value interface{})
```

## Recording user IDs with the X\-Ray SDK for Go<a name="xray-sdk-go-segment-userid"></a>

Record user IDs on request segments to identify the user who sent the request\.

**To record user IDs**

1. Get a reference to the current segment from `AWSXRay`\.

   ```
   import (
     "context"
     "github.com/aws/aws-xray-sdk-go/xray"
   )
   
   mySegment := xray.GetSegment(context)
   ```

1. Call `setUser` with a String ID of the user who sent the request\.

   ```
   mySegment.User = "U12345"
   ```

To find traces for a user ID, use the `user` keyword in a [filter expression](xray-console-filters.md)\.