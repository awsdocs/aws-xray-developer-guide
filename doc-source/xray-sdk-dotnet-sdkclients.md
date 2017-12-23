# Instrumenting Downstream Calls to AWS Services<a name="xray-sdk-dotnet-sdkclients"></a>

You can instrument your AWS SDK for \.NET clients by adding an event handler with `AWSSdkTracingHandler.AddEventHandler`\.

**Example SampleController\.cs \- DynamoDB Client Instrumentation**  
Initialize a DynamoDB client with the AWS SDK for \.NET, and create an `AWSSdkTracingHandler`, passing `AWSXRayRecorder.Instance` to the constructor\. Then call `AddEventHandler` on the SDK tracing handler\.  

```
using Amazon;
using Amazon.Util;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DocumentModel;
using [Amazon\.XRay\.Recorder\.Core](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
using [Amazon\.XRay\.Recorder\.Handlers\.AwsSdk](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_AwsSdk.htm);

namespace SampleEBWebApplication.Controllers
{
  public class SampleController : ApiController
  {
    private static readonly Lazy<AmazonDynamoDBClient> LazyDdbClient = new Lazy<AmazonDynamoDBClient>(() =>
    {
      var client = new AmazonDynamoDBClient(EC2InstanceMetadata.Region ?? RegionEndpoint.USEast1);
      new AWSSdkTracingHandler(AWSXRayRecorder.Instance).AddEventHandler(client);
      return client;
    });
```

For all services, you can see the name of the API called in the X\-Ray console\. For a subset of services, the X\-Ray SDK adds information to the segment to provide more granularity in the service map\.

For example, when you make a call with an instrumented DynamoDB client, the SDK adds the table name to the segment for calls that target a table\. In the console, each table appears as a separate node in the service map, with a generic DynamoDB node for calls that don't target a table\.

**Example Subsegment for a Call to DynamoDB to Save an Item**  

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

+ **Amazon DynamoDB** – table name

+ **Amazon Simple Storage Service** – bucket and key name

+ **Amazon Simple Queue Service** – queue name