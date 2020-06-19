# Add annotations and metadata to segments with the X\-Ray SDK for Ruby<a name="xray-sdk-ruby-segment"></a>

You can record additional information about requests, the environment, or your application with annotations and metadata\. You can add annotations and metadata to the segments that the X\-Ray SDK creates, or to custom subsegments that you create\.

**Annotations** are key\-value pairs with string, number, or Boolean values\. Annotations are indexed for use with [filter expressions](xray-console-filters.md)\. Use annotations to record data that you want to use to group traces in the console, or when calling the [https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html](https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html) API\.

**Metadata** are key\-value pairs that can have values of any type, including objects and lists, but are not indexed for use with filter expressions\. Use metadata to record additional data that you want stored in the trace but don't need to use with search\.

In addition to annotations and metadata, you can also [record user ID strings](#xray-sdk-ruby-segment-userid) on segments\. User IDs are recorded in a separate field on segments and are indexed for use with search\.

**Topics**
+ [Recording annotations with the X\-Ray SDK for Ruby](#xray-sdk-ruby-segment-annotations)
+ [Recording metadata with the X\-Ray SDK for Ruby](#xray-sdk-ruby-segment-metadata)
+ [Recording user IDs with the X\-Ray SDK for Ruby](#xray-sdk-ruby-segment-userid)

## Recording annotations with the X\-Ray SDK for Ruby<a name="xray-sdk-ruby-segment-annotations"></a>

Use annotations to record information on segments or subsegments that you want indexed for search\.

**Annotation Requirements**
+ **Keys** – Up to 500 alphanumeric characters\. No spaces or symbols except underscores\.
+ **Values** – Up to 1,000 Unicode characters\.
+ **Entries** – Up to 50 annotations per trace\.

**To record annotations**

1. Get a reference to the current segment or subsegment from `xray_recorder`\.

   ```
   require 'aws-xray-sdk'
   ...
   document = XRay.recorder.current_segment
   ```

   or

   ```
   require 'aws-xray-sdk'
   ...
   document = XRay.recorder.current_subsegment
   ```

1. Call `update` with a hash value\.

   ```
   my_annotations = { id: 12345 }
   document.annotations.update my_annotations
   ```

The SDK records annotations as key\-value pairs in an `annotations` object in the segment document\. Calling `add_annotations` twice with the same key overwrites previously recorded values on the same segment or subsegment\.

To find traces that have annotations with specific values, use the `annotations.key` keyword in a [filter expression](xray-console-filters.md)\.

## Recording metadata with the X\-Ray SDK for Ruby<a name="xray-sdk-ruby-segment-metadata"></a>

Use metadata to record information on segments or subsegments that you don't need indexed for search\. Metadata values can be strings, numbers, Booleans, or any object that can be serialized into a JSON object or array\.

**To record metadata**

1. Get a reference to the current segment or subsegment from `xray_recorder`\.

   ```
   require 'aws-xray-sdk'
   ...
   document = XRay.recorder.current_segment
   ```

   or

   ```
   require 'aws-xray-sdk'
   ...
   document = XRay.recorder.current_subsegment
   ```

1. Call `metadata` with a String key; a Boolean, Number, String, or Object value; and a String namespace\.

   ```
   my_metadata = {
     my_namespace: {
       key: 'value'
     }
   }
   subsegment.metadata my_metadata
   ```

Calling `metadata` twice with the same key overwrites previously recorded values on the same segment or subsegment\.

## Recording user IDs with the X\-Ray SDK for Ruby<a name="xray-sdk-ruby-segment-userid"></a>

Record user IDs on request segments to identify the user who sent the request\.

**To record user IDs**

1. Get a reference to the current segment from `xray_recorder`\.

   ```
   require 'aws-xray-sdk'
   ...
   document = XRay.recorder.current_segment
   ```

1. Set the user field on the segment to a String ID of the user who sent the request\.

   ```
   segment.user = 'U12345'
   ```

You can set the user in your controllers to record the user ID as soon as your application starts processing a request\.

To find traces for a user ID, use the `user` keyword in a [filter expression](xray-console-filters.md)\.