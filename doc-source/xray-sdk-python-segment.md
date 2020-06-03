# Add annotations and metadata to segments with the X\-Ray SDK for Python<a name="xray-sdk-python-segment"></a>

You can record additional information about requests, the environment, or your application with annotations and metadata\. You can add annotations and metadata to the segments that the X\-Ray SDK creates, or to custom subsegments that you create\.

**Annotations** are key\-value pairs with string, number, or Boolean values\. Annotations are indexed for use with [filter expressions](xray-console-filters.md)\. Use annotations to record data that you want to use to group traces in the console, or when calling the [https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html](https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html) API\.

**Metadata** are key\-value pairs that can have values of any type, including objects and lists, but are not indexed for use with filter expressions\. Use metadata to record additional data that you want stored in the trace but don't need to use with search\.

In addition to annotations and metadata, you can also [record user ID strings](#xray-sdk-python-segment-userid) on segments\. User IDs are recorded in a separate field on segments and are indexed for use with search\.

**Topics**
+ [Recording annotations with the X\-Ray SDK for Python](#xray-sdk-python-segment-annotations)
+ [Recording metadata with the X\-Ray SDK for Python](#xray-sdk-python-segment-metadata)
+ [Recording user IDs with the X\-Ray SDK for Python](#xray-sdk-python-segment-userid)

## Recording annotations with the X\-Ray SDK for Python<a name="xray-sdk-python-segment-annotations"></a>

Use annotations to record information on segments or subsegments that you want indexed for search\.

**Annotation Requirements**
+ **Keys** – Up to 500 alphanumeric characters\. No spaces or symbols except underscores\.
+ **Values** – Up to 1,000 Unicode characters\.
+ **Entries** – Up to 50 annotations per trace\.

**To record annotations**

1. Get a reference to the current segment or subsegment from `xray_recorder`\.

   ```
   from aws_xray_sdk.core import xray_recorder
   ...
   document = xray_recorder.current_segment()
   ```

   or

   ```
   from aws_xray_sdk.core import xray_recorder
   ...
   document = xray_recorder.current_subsegment()
   ```

1. Call `put_annotation` with a String key, and a Boolean, Number, or String value\.

   ```
   document.put_annotation("mykey", "my value");
   ```

Alternatively, you can use the `put_annotation` method on the `xray_recorder`\. This method records annotations on the current subsegment or, if no subsegment is open, on the segment\.

```
xray_recorder.put_annotation("mykey", "my value");
```

The SDK records annotations as key\-value pairs in an `annotations` object in the segment document\. Calling `put_annotation` twice with the same key overwrites previously recorded values on the same segment or subsegment\.

To find traces that have annotations with specific values, use the `annotations.key` keyword in a [filter expression](xray-console-filters.md)\.

## Recording metadata with the X\-Ray SDK for Python<a name="xray-sdk-python-segment-metadata"></a>

Use metadata to record information on segments or subsegments that you don't need indexed for search\. Metadata values can be strings, numbers, Booleans, or any object that can be serialized into a JSON object or array\.

**To record metadata**

1. Get a reference to the current segment or subsegment from `xray_recorder`\.

   ```
   from aws_xray_sdk.core import xray_recorder
   ...
   document = xray_recorder.current_segment()
   ```

   or

   ```
   from aws_xray_sdk.core import xray_recorder
   ...
   document = xray_recorder.current_subsegment()
   ```

1. Call `put_metadata` with a String key; a Boolean, Number, String, or Object value; and a String namespace\.

   ```
   document.put_metadata("my key", "my value", "my namespace");
   ```

   or

   Call `put_metadata` with just a key and value\.

   ```
   document.put_metadata("my key", "my value");
   ```

Alternatively, you can use the `put_metadata` method on the `xray_recorder`\. This method records metadata on the current subsegment or, if no subsegment is open, on the segment\.

```
xray_recorder.put_metadata("my key", "my value");
```

If you don't specify a namespace, the SDK uses `default`\. Calling `put_metadata` twice with the same key overwrites previously recorded values on the same segment or subsegment\.

## Recording user IDs with the X\-Ray SDK for Python<a name="xray-sdk-python-segment-userid"></a>

Record user IDs on request segments to identify the user who sent the request\.

**To record user IDs**

1. Get a reference to the current segment from `xray_recorder`\.

   ```
   from aws_xray_sdk.core import xray_recorder
   ...
   document = xray_recorder.current_segment()
   ```

1. Call `setUser` with a String ID of the user who sent the request\.

   ```
   document.set_user("U12345");
   ```

You can call `set_user` in your controllers to record the user ID as soon as your application starts processing a request\.

To find traces for a user ID, use the `user` keyword in a [filter expression](xray-console-filters.md)\.