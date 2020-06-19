# Tracing AWS SDK calls with the X\-Ray SDK for Go<a name="xray-sdk-go-awssdkclients"></a>

When your application makes calls to AWS services to store data, write to a queue, or send notifications, the X\-Ray SDK for Go tracks the calls downstream in [subsegments](xray-sdk-go-subsegments.md)\. Traced AWS services and resources that you access within those services \(for example, an Amazon S3 bucket or Amazon SQS queue\), appear as downstream nodes on the service map in the X\-Ray console\.

To trace AWS SDK clients, wrap the client object with the `xray.AWS()` call as shown in the following example\.

**Example main\.go**  

```
var dynamo *dynamodb.DynamoDB
func main() {
  dynamo = dynamodb.New(session.Must(session.NewSession()))
  xray.AWS(dynamo.Client)
}
```

Then, when you use the AWS SDK client, use the `withContext` version of the call method, and pass it the `context` from the `http.Request` object passed to the [handler](xray-sdk-go-handler.md)\.

**Example main\.go – AWS SDK call**  

```
func listTablesWithContext(ctx context.Context) {
  output := dynamo.ListTablesWithContext(ctx, &dynamodb.ListTablesInput{})
  doSomething(output)
}
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