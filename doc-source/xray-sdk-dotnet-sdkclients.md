# Tracing AWS SDK calls with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-sdkclients"></a>

When your application makes calls to AWS services to store data, write to a queue, or send notifications, the X\-Ray SDK for \.NET tracks the calls downstream in [subsegments](xray-sdk-dotnet-subsegments.md)\. Traced AWS services and resources that you access within those services \(for example, an Amazon S3 bucket or Amazon SQS queue\), appear as downstream nodes on the service map in the X\-Ray console\.

You can instrument all of your AWS SDK for \.NET clients by calling `RegisterXRayForAllServices` before you create them\.

**Example SampleController\.cs \- DynamoDB client instrumentation**  

```
using Amazon;
using Amazon.Util;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DocumentModel;
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
using [Amazon\.XRay\.Recorder\.Handlers\.AwsSdk](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_AwsSdk.htm);

namespace SampleEBWebApplication.Controllers
{
  public class SampleController : ApiController
  {
    AWSSDKHandler.RegisterXRayForAllServices();
    private static readonly Lazy<AmazonDynamoDBClient> LazyDdbClient = new Lazy<AmazonDynamoDBClient>(() =>
    {
      var client = new AmazonDynamoDBClient(EC2InstanceMetadata.Region ?? RegionEndpoint.USEast1);
      return client;
    });
```

To instrument clients for some services and not others, call `RegisterXRay` instead of `RegisterXRayForAllServices`\. Replace the highlighted text with the name of the service's client interface\.

```
AWSSDKHandler.RegisterXRay<IAmazonDynamoDB>()
```

For all services, you can see the name of the API called in the X\-Ray console\. For a subset of services, the X\-Ray SDK adds information to the segment to provide more granularity in the service map\.

For example, when you make a call with an instrumented DynamoDB client, the SDK adds the table name to the segment for calls that target a table\. In the console, each table appears as a separate node in the service map, with a generic DynamoDB node for calls that don't target a table\.

**Example Subsegment for a call to DynamoDB to save an item**  

```
{
  "id": "24756640c0d0978a",
  "start_time": 1.480305974194E9,
  "end_time": 1.4803059742E9,
  "name": "DynamoDB",
  "namespace": "aws",
  "http": {
    "response": {
      "content_length": 60,
      "status": 200
    }
  },
  "aws": {
    "table_name": "scorekeep-user",
    "operation": "UpdateItem",
    "request_id": "UBQNSO5AEM8T4FDA4RQDEB94OVTDRVV4K4HIRGVJF66Q9ASUAAJG",
  }
}
```

When you access named resources, calls to the following services create additional nodes in the service map\. Calls that don't target specific resources create a generic node for the service\.
+ **Amazon DynamoDB** – Table name
+ **Amazon Simple Storage Service** – Bucket and key name
+ **Amazon Simple Queue Service** – Queue name