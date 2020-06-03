# Tracing AWS SDK calls with the X\-Ray SDK for Java<a name="xray-sdk-java-awssdkclients"></a>

When your application makes calls to AWS services to store data, write to a queue, or send notifications, the X\-Ray SDK for Java tracks the calls downstream in [subsegments](xray-sdk-java-subsegments.md)\. Traced AWS services and resources that you access within those services \(for example, an Amazon S3 bucket or Amazon SQS queue\), appear as downstream nodes on the service map in the X\-Ray console\.

The X\-Ray SDK for Java automatically instruments all AWS SDK clients when you include the `aws-sdk` and an `aws-sdk-instrumentor` [submodules](xray-sdk-java.md#xray-sdk-java-submodules) in your build\. If you don't include the Instrumentor submodule, you can choose to instrument some clients while excluding others\.

To instrument individual clients, remove the `aws-sdk-instrumentor` submodule from your build and add an `XRayClient` as a `TracingHandler` on your AWS SDK client using the service's client builder\.

For example, to instrument an `AmazonDynamoDB` client, pass a tracing handler to `AmazonDynamoDBClientBuilder`\.

**Example MyModel\.java \- DynamoDB client**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.handlers\.TracingHandler](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/handlers/TracingHandler.html);

...
public class MyModel {
  private AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
        .withRegion(Regions.fromName(System.getenv("AWS_REGION")))
        .withRequestHandlers(new TracingHandler(AWSXRay.getGlobalRecorder()))
        .build();
...
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

To instrument downstream calls to AWS services with AWS SDK for Java 2\.2 and later, you can omit the `aws-xray-recorder-sdk-aws-sdk-v2-instrumentor` module from your build configuration\. Include the `aws-xray-recorder-sdk-aws-sdk-v2 module` instead, then instrument individual clients by configuring them with a `TracingInterceptor`\. 

**Example AWS SDK for Java 2\.2 and later \- tracing interceptor**  

```
import com.amazonaws.xray.interceptors.TracingInterceptor;
import software.amazon.awssdk.core.client.config.ClientOverrideConfiguration
import software.amazon.awssdk.services.dynamodb.DynamoDbClient;
//...
public class MyModel {
private DynamoDbClient client = DynamoDbClient.builder()
.region(Region.US_WEST_2)
.overrideConfiguration(ClientOverrideConfiguration.builder()
.addExecutionInterceptor(new TracingInterceptor())
.build()
)
.build();
//...
```