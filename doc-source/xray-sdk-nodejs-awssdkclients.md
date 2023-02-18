# Tracing AWS SDK calls with the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-awssdkclients"></a>

When your application makes calls to AWS services to store data, write to a queue, or send notifications, the X\-Ray SDK for Node\.js tracks the calls downstream in [subsegments](xray-sdk-nodejs-subsegments.md)\. Traced AWS services, and resources that you access within those services \(for example, an Amazon S3 bucket or Amazon SQS queue\), appear as downstream nodes on the service map in the X\-Ray console\.

Instrument AWS SDK clients that you create via the [AWS SDK for JavaScript V2](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/welcome.html) or [AWS SDK for JavaScript V3](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/welcome.html)\. Each AWS SDK version provides different methods for instrumenting AWS SDK clients\.

**Note**  
Currently, the AWS X\-Ray SDK for Node\.js returns less segment information when instrumenting AWS SDK for JavaScript V3 clients, as compared to instrumenting V2 clients\. For instance, subsegments representing calls to DynamoDB will not return the table name\. If you need this segment information in your traces, consider using the AWS SDK for JavaScript V2\. 

------
#### [ AWS SDK for JavaScript V2 ]

You can instrument all AWS SDK V2 clients by wrapping your `aws-sdk` require statement in a call to `AWSXRay.captureAWS`\.

**Example app\.js \- AWS SDK instrumentation**  

```
const AWS = AWSXRay.captureAWS(require('aws-sdk'));
```

To instrument individual clients, wrap your AWS SDK client in a call to `AWSXRay.captureAWSClient`\. For example, to instrument an `AmazonDynamoDB` client:

**Example app\.js \- DynamoDB client instrumentation**  

```
    const AWSXRay = require('aws-xray-sdk');
...
    const ddb = AWSXRay.captureAWSClient(new AWS.DynamoDB());
```

**Warning**  
Do not use both `captureAWS` and `captureAWSClient` together\. This will lead to duplicate subsegments\.

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

------
#### [ AWS SDK for JavaScript V3 ]

The AWS SDK for JavaScript V3 is modular, so your code only loads the modules it needs\. Because of this, it isn't possible to instrument all AWS SDK clients as V3 does not support the `captureAWS` method\. Instrument each AWS SDK client using the `AWSXRay.captureAWSv3Client` method\. For example, to instrument an `AmazonDynamoDB` client:

**Example app\.js \- DynamoDB client instrumentation using SDK for Javascript V3**  

```
    const AWSXRay = require('aws-xray-sdk');
    const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
...
    const ddb = AWSXRay.captureAWSv3Client(new DynamoDBClient({ region: "region" }));
```

When using AWS SDK for JavaScript V3, metadata such as table name, bucket and key name, or queue name, are not currently returned, and therefore the service map will not contain discrete nodes for each named resource as it would when instrumenting AWS SDK clients using the AWS SDK for JavaScript V2\.

**Example Subsegment for a call to DynamoDB to save an item, when using the AWS SDK for JavaScript V3**  

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
    "operation": "UpdateItem",
    "request_id": "UBQNSO5AEM8T4FDA4RQDEB94OVTDRVV4K4HIRGVJF66Q9ASUAAJG",
  }
}
```

------