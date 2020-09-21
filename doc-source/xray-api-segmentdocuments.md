# AWS X\-Ray segment documents<a name="xray-api-segmentdocuments"></a>

A **trace segment** is a JSON representation of a request that your application serves\. A trace segment records information about the original request, information about the work that your application does locally, and **subsegments** with information about downstream calls that your application makes to AWS resources, HTTP APIs, and SQL databases\.

A **segment document** conveys information about a segment to X\-Ray\. A segment document can be up to 64 kB and contain a whole segment with subsegments, a fragment of a segment that indicates that a request is in progress, or a single subsegment that is sent separately\. You can send segment documents directly to X\-Ray by using the [https://docs.aws.amazon.com/xray/latest/api/API_PutTraceSegments.html](https://docs.aws.amazon.com/xray/latest/api/API_PutTraceSegments.html) API\.

X\-Ray compiles and processes segment documents to generate queryable **trace summaries** and **full traces** that you can access by using the [https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html](https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html) and [https://docs.aws.amazon.com/xray/latest/api/API_BatchGetTraces.html](https://docs.aws.amazon.com/xray/latest/api/API_BatchGetTraces.html) APIs, respectively\. In addition to the segments and subsegments that you send to X\-Ray, the service uses information in subsegments to generate **inferred segments** and adds them to the full trace\. Inferred segments represent downstream services and resources in the service map\.

X\-Ray provides a **JSON schema** for segment documents\. You can download the schema here: [xray\-segmentdocument\-schema\-v1\.0\.0](samples/xray-segmentdocument-schema-v1.0.0.zip)\. The fields and objects listed in the schema are described in more detail in the following sections\.

A subset of segment fields are indexed by X\-Ray for use with filter expressions\. For example, if you set the `user` field on a segment to a unique identifier, you can search for segments associated with specific users in the X\-Ray console or by using the `GetTraceSummaries` API\. For more information, see [Using filter expressions to search for traces in the console](xray-console-filters.md)\.

When you instrument your application with the X\-Ray SDK, the SDK generates segment documents for you\. Instead of sending segment documents directly to X\-Ray, the SDK transmits them over a local UDP port to the [X\-Ray daemon](xray-daemon.md)\. For more information, see [Sending segment documents to the X\-Ray daemon](xray-api-sendingdata.md#xray-api-daemon)\.

**Topics**
+ [Segment fields](#api-segmentdocuments-fields)
+ [Subsegments](#api-segmentdocuments-subsegments)
+ [HTTP request data](#api-segmentdocuments-http)
+ [Annotations](#api-segmentdocuments-annotations)
+ [Metadata](#api-segmentdocuments-metadata)
+ [AWS resource data](#api-segmentdocuments-aws)
+ [Errors and exceptions](#api-segmentdocuments-errors)
+ [SQL queries](#api-segmentdocuments-sql)

## Segment fields<a name="api-segmentdocuments-fields"></a>

A segment records tracing information about a request that your application serves\. At a minimum, a segment records the name, ID, start time, trace ID, and end time of the request\.

**Example Minimal complete segment**  

```
{
  "name" : "example.com",
  "id" : "70de5b6f19ff9a0a",
  "start_time" : 1.478293361271E9,
  "trace_id" : "1-581cf771-a006649127e371903a2de979",
  "end_time" : 1.478293361449E9
}
```

The following fields are required, or conditionally required, for segments\.

**Note**  
Values must be strings \(up to 250 characters\) unless noted otherwise\.

**Required Segment Fields**
+ `name` – The logical name of the service that handled the request, up to **200 characters**\. For example, your application's name or domain name\. Names can contain Unicode letters, numbers, and whitespace, and the following symbols: `_`, `.`, `:`, `/`, `%`, `&`, `#`, `=`, `+`, `\`, `-`, `@`
+ `id` – A 64\-bit identifier for the segment, unique among segments in the same trace, in **16 hexadecimal digits**\.
+ `trace_id` – A unique identifier that connects all segments and subsegments originating from a single client request\.

**Trace ID Format**

  A `trace_id` consists of three numbers separated by hyphens\. For example, `1-58406520-a006649127e371903a2de979`\. This includes:
  + The version number, that is, `1`\.
  + The time of the original request, in Unix epoch time, in **8 hexadecimal digits**\.

    For example, 10:00AM December 1st, 2016 PST in epoch time is `1480615200` seconds, or `58406520` in hexadecimal digits\.
  + A 96\-bit identifier for the trace, globally unique, in **24 hexadecimal digits**\.
**Trace ID Security**  
Trace IDs are visible in [response headers](xray-concepts.md#xray-concepts-tracingheader)\. Generate trace IDs with a secure random algorithm to ensure that attackers cannot calculate future trace IDs and send requests with those IDs to your application\.
+ `start_time` – **number** that is the time the segment was created, in floating point seconds in epoch time\. For example, `1480615200.010` or `1.480615200010E9`\. Use as many decimal places as you need\. Microsecond resolution is recommended when available\.
+ `end_time` – **number** that is the time the segment was closed\. For example, `1480615200.090` or `1.480615200090E9`\. Specify either an `end_time` or `in_progress`\.
+ `in_progress` – **boolean**, set to `true` instead of specifying an `end_time` to record that a segment is started, but is not complete\. Send an in\-progress segment when your application receives a request that will take a long time to serve, to trace the request receipt\. When the response is sent, send the complete segment to overwrite the in\-progress segment\. Only send one complete segment, and one or zero in\-progress segments, per request\.

**Service Names**  
A segment's `name` should match the domain name or logical name of the service that generates the segment\. However, this is not enforced\. Any application that has permission to [https://docs.aws.amazon.com/xray/latest/api/API_PutTraceSegments.html](https://docs.aws.amazon.com/xray/latest/api/API_PutTraceSegments.html) can send segments with any name\.

The following fields are optional for segments\.

**Optional Segment Fields**
+ `service` – An object with information about your application\.
  + `version` – A string that identifies the version of your application that served the request\.
+ `user` – A string that identifies the user who sent the request\.
+ `origin` – The type of AWS resource running your application\.

**Supported Values**
  + `AWS::EC2::Instance` – An Amazon EC2 instance\.
  + `AWS::ECS::Container` – An Amazon ECS container\.
  + `AWS::ElasticBeanstalk::Environment` – An Elastic Beanstalk environment\.

  When multiple values are applicable to your application, use the one that is most specific\. For example, a Multicontainer Docker Elastic Beanstalk environment runs your application on an Amazon ECS container, which in turn runs on an Amazon EC2 instance\. In this case you would set the origin to `AWS::ElasticBeanstalk::Environment` as the environment is the parent of the other two resources\.
+ `parent_id` – A subsegment ID you specify if the request originated from an instrumented application\. The X\-Ray SDK adds the parent subsegment ID to the [tracing header](xray-concepts.md#xray-concepts-tracingheader) for downstream HTTP calls\. In the case of nested subsegments, a subsegment can have a segment or a subsegment as its parent\. 
+ `http` – [`http`](#api-segmentdocuments-http) objects with information about the original HTTP request\.
+ `aws` – [`aws`](#api-segmentdocuments-aws) object with information about the AWS resource on which your application served the request\.
+ `error`, `throttle`, `fault`, and `cause` – [error](#api-segmentdocuments-errors) fields that indicate an error occurred and that include information about the exception that caused the error\.
+ `annotations` – [`annotations`](#api-segmentdocuments-annotations) object with key\-value pairs that you want X\-Ray to index for search\.
+ `metadata` – [`metadata`](#api-segmentdocuments-metadata) object with any additional data that you want to store in the segment\.
+ `subsegments` – **array** of [`subsegment`](#api-segmentdocuments-subsegments) objects\.

## Subsegments<a name="api-segmentdocuments-subsegments"></a>

You can create subsegments to record calls to AWS services and resources that you make with the AWS SDK, calls to internal or external HTTP web APIs, or SQL database queries\. You can also create subsegments to debug or annotate blocks of code in your application\. Subsegments can contain other subsegments, so a custom subsegment that records metadata about an internal function call can contain other custom subsegments and subsegments for downstream calls\.

A subsegment records a downstream call from the point of view of the service that calls it\. X\-Ray uses subsegments to identify downstream services that don't send segments and create entries for them on the service graph\.

A subsegment can be embedded in a full segment document or sent independently\. Send subsegments separately to asynchronously trace downstream calls for long\-running requests, or to avoid exceeding the maximum segment document size\.

**Example Segment with embedded subsegment**  
An independent subsegment has a `type` of `subsegment` and a `parent_id` that identifies the parent segment\.  

```
{
  "trace_id"   : "1-5759e988-bd862e3fe1be46a994272793",
  "id"         : "defdfd9912dc5a56",
  "start_time" : 1461096053.37518,
  "end_time"   : 1461096053.4042,
  "name"       : "www.example.com",
  "http"       : {
    "request"  : {
      "url"        : "https://www.example.com/health",
      "method"     : "GET",
      "user_agent" : "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/601.7.7",
      "client_ip"  : "11.0.3.111"
    },
    "response" : {
      "status"         : 200,
      "content_length" : 86
    }
  },
  "subsegments" : [
    {
      "id"         : "53995c3f42cd8ad8",
      "name"       : "api.example.com",
      "start_time" : 1461096053.37769,
      "end_time"   : 1461096053.40379,
      "namespace"  : "remote",
      "http"       : {
        "request"  : {
          "url"    : "https://api.example.com/health",
          "method" : "POST",
          "traced" : true
        },
        "response" : {
          "status"         : 200,
          "content_length" : 861
        }
      }
    }
  ]
}
```

For long\-running requests, you can send an in\-progress segment to notify X\-Ray that the request was received, and then send subsegments separately to trace them before completing the original request\.

**Example In\-progress segment**  

```
{
  "name" : "example.com",
  "id" : "70de5b6f19ff9a0b",
  "start_time" : 1.478293361271E9,
  "trace_id" : "1-581cf771-a006649127e371903a2de979",
  "in_progress": true
}
```

**Example Independent subsegment**  
An independent subsegment has a `type` of `subsegment`, a `trace_id`, and a `parent_id` that identifies the parent segment\.  

```
{
  "name" : "api.example.com",
  "id" : "53995c3f42cd8ad8",
  "start_time" : 1.478293361271E9,
  "end_time" : 1.478293361449E9,
  "type" : "subsegment",
  "trace_id" : "1-581cf771-a006649127e371903a2de979"
  "parent_id" : "defdfd9912dc5a56",
  "namespace"  : "remote",
  "http"       : {
      "request"  : {
          "url"    : "https://api.example.com/health",
          "method" : "POST",
          "traced" : true
      },
      "response" : {
          "status"         : 200,
          "content_length" : 861
      }
  }
}
```

When the request is complete, close the segment by resending it with an `end_time`\. The complete segment overwrites the in\-progress segment\.

You can also send subsegments separately for completed requests that triggered asynchronous workflows\. For example, a web API may return a `OK 200` response immediately prior to starting the work that the user requested\. You can send a full segment to X\-Ray as soon as the response is sent, followed by subsegments for work completed later\. As with segments, you can also send a subsegment fragment to record that the subsegment has started, and then overwrite it with a full subsegment once the downstream call is complete\.

The following fields are required, or are conditionally required, for subsegments\.

**Note**  
Values are strings up to 250 characters unless noted otherwise\.

**Required Subsegment Fields**
+ `id` – A 64\-bit identifier for the subsegment, unique among segments in the same trace, in **16 hexadecimal digits**\.
+ `name` – The logical name of the subsegment\. For downstream calls, name the subsegment after the resource or service called\. For custom subsegments, name the subsegment after the code that it instruments \(e\.g\., a function name\)\.
+ `start_time` – **number** that is the time the subsegment was created, in floating point seconds in epoch time, accurate to milliseconds\. For example, `1480615200.010` or `1.480615200010E9`\.
+ `end_time` – **number** that is the time the subsegment was closed\. For example, `1480615200.090` or `1.480615200090E9`\. Specify an `end_time` or `in_progress`\.
+ `in_progress` – **boolean** that is set to `true` instead of specifying an `end_time` to record that a subsegment is started, but is not complete\. Only send one complete subsegment, and one or zero in\-progress subsegments, per downstream request\.
+ `trace_id` – Trace ID of the subsegment's parent segment\. Required only if sending a subsegment separately\.

**Trace ID Format**

  A `trace_id` consists of three numbers separated by hyphens\. For example, `1-58406520-a006649127e371903a2de979`\. This includes:
  + The version number, that is, `1`\.
  + The time of the original request, in Unix epoch time, in **8 hexadecimal digits**\.

    For example, 10:00AM December 1st, 2016 PST in epoch time is `1480615200` seconds, or `58406520` in hexadecimal digits\.
  + A 96\-bit identifier for the trace, globally unique, in **24 hexadecimal digits**\.
+ `parent_id` – Segment ID of the subsegment's parent segment\. Required only if sending a subsegment separately\. In the case of nested subsegments, a subsegment can have a segment or a subsegment as its parent\.
+ `type` – `subsegment`\. Required only if sending a subsegment separately\.

The following fields are optional for subsegments\.

**Optional Subsegment Fields**
+ `namespace` – `aws` for AWS SDK calls; `remote` for other downstream calls\.
+ `http` – [`http`](#api-segmentdocuments-http) object with information about an outgoing HTTP call\.
+ `aws` – [`aws`](#api-segmentdocuments-aws) object with information about the downstream AWS resource that your application called\.
+ `error`, `throttle`, `fault`, and `cause` – [error](#api-segmentdocuments-errors) fields that indicate an error occurred and that include information about the exception that caused the error\.
+ `annotations` – [`annotations`](#api-segmentdocuments-annotations) object with key\-value pairs that you want X\-Ray to index for search\.
+ `metadata` – [`metadata`](#api-segmentdocuments-metadata) object with any additional data that you want to store in the segment\.
+ `subsegments` – **array** of [`subsegment`](#api-segmentdocuments-subsegments) objects\.
+ `precursor_ids` – **array** of subsegment IDs that identifies subsegments with the same parent that completed prior to this subsegment\.

## HTTP request data<a name="api-segmentdocuments-http"></a>

Use an HTTP block to record details about an HTTP request that your application served \(in a segment\) or that your application made to a downstream HTTP API \(in a subsegment\)\. Most of the fields in this object map to information found in an HTTP request and response\.

**`http`**

All fields are optional\.
+ `request` – Information about a request\.
  + `method` – The request method\. For example, `GET`\.
  + `url` – The full URL of the request, compiled from the protocol, hostname, and path of the request\.
  + `user_agent` – The user agent string from the requester's client\.
  + `client_ip` – The IP address of the requester\. Can be retrieved from the IP packet's `Source Address` or, for forwarded requests, from an `X-Forwarded-For` header\.
  + `x_forwarded_for` – \(segments only\) **boolean** indicating that the `client_ip` was read from an `X-Forwarded-For` header and is not reliable as it could have been forged\.
  + `traced` – \(subsegments only\) **boolean** indicating that the downstream call is to another traced service\. If this field is set to `true`, X\-Ray considers the trace to be broken until the downstream service uploads a segment with a `parent_id` that matches the `id` of the subsegment that contains this block\.
+ `response` – Information about a response\.
  + `status` – **number** indicating the HTTP status of the response\.
  + `content_length` – **number** indicating the length of the response body in bytes\.

When you instrument a call to a downstream web api, record a subsegment with information about the HTTP request and response\. X\-Ray uses the subsegment to generate an inferred segment for the remote API\.

**Example Segment for HTTP call served by an application running on Amazon EC2**  

```
{
  "id": "6b55dcc497934f1a",
  "start_time": 1484789387.126,
  "end_time": 1484789387.535,
  "trace_id": "1-5880168b-fd5158284b67678a3bb5a78c",
  "name": "www.example.com",
  "origin": "AWS::EC2::Instance",
  "aws": {
    "ec2": {
      "availability_zone": "us-west-2c",
      "instance_id": "i-0b5a4678fc325bg98"
    },
    "xray": {
        "sdk_version": "2.4.0 for Java"
    },
  },
  "http": {
    "request": {
      "method": "POST",
      "client_ip": "78.255.233.48",
      "url": "http://www.example.com/api/user",
      "user_agent": "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:45.0) Gecko/20100101 Firefox/45.0",
      "x_forwarded_for": true
    },
    "response": {
      "status": 200
    }
  }
```

**Example Subsegment for a downstream HTTP call**  

```
{
  "id": "004f72be19cddc2a",
  "start_time": 1484786387.131,
  "end_time": 1484786387.501,
  "name": "names.example.com",
  "namespace": "remote",
  "http": {
    "request": {
      "method": "GET",
      "url": "https://names.example.com/"
    },
    "response": {
      "content_length": -1,
      "status": 200
    }
  }
}
```

**Example Inferred segment for a downstream HTTP call**  

```
{
  "id": "168416dc2ea97781",
  "name": "names.example.com",
  "trace_id": "1-5880168b-fd5153bb58284b67678aa78c",
  "start_time": 1484786387.131,
  "end_time": 1484786387.501,
  "parent_id": "004f72be19cddc2a",
  "http": {
    "request": {
      "method": "GET",
      "url": "https://names.example.com/"
    },
    "response": {
      "content_length": -1,
      "status": 200
    }
  },
  "inferred": true
}
```

## Annotations<a name="api-segmentdocuments-annotations"></a>

Segments and subsegments can include an `annotations` object containing one or more fields that X\-Ray indexes for use with filter expressions\. Fields can have string, number, or Boolean values \(no objects or arrays\)\. X\-Ray indexes up to 50 annotations per trace\.

**Example Segment for HTTP call with annotations**  

```
{
  "id": "6b55dcc497932f1a",
  "start_time": 1484789187.126,
  "end_time": 1484789187.535,
  "trace_id": "1-5880168b-fd515828bs07678a3bb5a78c",
  "name": "www.example.com",
  "origin": "AWS::EC2::Instance",
  "aws": {
    "ec2": {
      "availability_zone": "us-west-2c",
      "instance_id": "i-0b5a4678fc325bg98"
    },
    "xray": {
        "sdk_version": "2.4.0 for Java"
    },
  },
  "annotations": {
    "customer_category" : 124,
    "zip_code" : 98101,
    "country" : "United States",
    "internal" : false
  },
  "http": {
    "request": {
      "method": "POST",
      "client_ip": "78.255.233.48",
      "url": "http://www.example.com/api/user",
      "user_agent": "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:45.0) Gecko/20100101 Firefox/45.0",
      "x_forwarded_for": true
    },
    "response": {
      "status": 200
    }
  }
```

Keys must be alphanumeric in order to work with filters\. Underscore is allowed\. Other symbols and whitespace are not allowed\.

## Metadata<a name="api-segmentdocuments-metadata"></a>

Segments and subsegments can include a `metadata` object containing one or more fields with values of any type, including objects and arrays\. X\-Ray does not index metadata, and values can be any size, as long as the segment document doesn't exceed the maximum size \(64 kB\)\. You can view metadata in the full segment document returned by the [https://docs.aws.amazon.com/xray/latest/api/API_BatchGetTraces.html](https://docs.aws.amazon.com/xray/latest/api/API_BatchGetTraces.html) API\. Field keys \(`debug` in the following example\) starting with `AWS.` are reserved for use by AWS\-provided SDKs and clients\.

**Example Custom subsegment with metadata**  

```
{
  "id": "0e58d2918e9038e8",
  "start_time": 1484789387.502,
  "end_time": 1484789387.534,
  "name": "## UserModel.saveUser",
  "metadata": {
    "debug": {
      "test": "Metadata string from UserModel.saveUser"
    }
  },
  "subsegments": [
    {
      "id": "0f910026178b71eb",
      "start_time": 1484789387.502,
      "end_time": 1484789387.534,
      "name": "DynamoDB",
      "namespace": "aws",
      "http": {
        "response": {
          "content_length": 58,
          "status": 200
        }
      },
      "aws": {
        "table_name": "scorekeep-user",
        "operation": "UpdateItem",
        "request_id": "3AIENM5J4ELQ3SPODHKBIRVIC3VV4KQNSO5AEMVJF66Q9ASUAAJG",
        "resource_names": [
          "scorekeep-user"
        ]
      }
    }
  ]
}
```

## AWS resource data<a name="api-segmentdocuments-aws"></a>

For segments, the `aws` object contains information about the resource on which your application is running\. Multiple fields can apply to a single resource\. For example, an application running in a multicontainer Docker environment on Elastic Beanstalk could have information about the Amazon EC2 instance, the Amazon ECS container running on the instance, and the Elastic Beanstalk environment itself\.

**`aws` \(Segments\)**

All fields are optional\.
+ `account_id` – If your application sends segments to a different AWS account, record the ID of the account running your application\.
+ `ecs` – Information about an Amazon ECS container\.
  + `container` – The container ID of the container running your application\.
+ `ec2` – Information about an EC2 instance\.
  + `instance_id` – The instance ID of the EC2 instance\.
  + `availability_zone` – The Availability Zone in which the instance is running\.  
**Example AWS block with plugins**  

  ```
  "aws": {
    "elastic_beanstalk": {
      "version_label": "app-5a56-170119_190650-stage-170119_190650",
      "deployment_id": 32,
      "environment_name": "scorekeep"
    },
    "ec2": {
      "availability_zone": "us-west-2c",
      "instance_id": "i-075ad396f12bc325a"
    },
    "xray": {
      "sdk": "2.4.0 for Java"
    }
  }
  ```
+ `elastic_beanstalk` – Information about an Elastic Beanstalk environment\. You can find this information in a file named `/var/elasticbeanstalk/xray/environment.conf` on the latest Elastic Beanstalk platforms\.
  + `environment_name` – The name of the environment\.
  + `version_label` – The name of the application version that is currently deployed to the instance that served the request\.
  + `deployment_id` – **number** indicating the ID of the last successful deployment to the instance that served the request\.

For subsegments, record information about the AWS services and resources that your application accesses\. X\-Ray uses this information to create inferred segments that represent the downstream services in your service map\.

**`aws` \(Subsegments\)**

All fields are optional\.
+ `operation` – The name of the API action invoked against an AWS service or resource\.
+ `account_id` – If your application accesses resources in a different account, or sends segments to a different account, record the ID of the account that owns the AWS resource that your application accessed\.
+ `region` – If the resource is in a region different from your application, record the region\. For example, `us-west-2`\.
+ `request_id` – Unique identifier for the request\.
+ `queue_url` – For operations on an Amazon SQS queue, the queue's URL\.
+ `table_name` – For operations on a DynamoDB table, the name of the table\.

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

## Errors and exceptions<a name="api-segmentdocuments-errors"></a>

When an error occurs, you can record details about the error and exceptions that it generated\. Record errors in segments when your application returns an error to the user, and in subsegments when a downstream call returns an error\.

**error types**

Set one or more of the following fields to `true` to indicate that an error occurred\. Multiple types can apply if errors compound\. For example, a `429 Too Many Requests` error from a downstream call may cause your application to return `500 Internal Server Error`, in which case all three types would apply\.
+ `error` – **boolean** indicating that a client error occurred \(response status code was 4XX Client Error\)\.
+ `throttle` – **boolean** indicating that a request was throttled \(response status code was *429 Too Many Requests*\)\.
+ `fault` – **boolean** indicating that a server error occurred \(response status code was 5XX Server Error\)\.

Indicate the cause of the error by including a **cause** object in the segment or subsegment\.

**`cause`**

A cause can be either a **16 character** exception ID or an object with the following fields:
+ `working_directory` – The full path of the working directory when the exception occurred\.
+ `paths` – The **array** of paths to libraries or modules in use when the exception occurred\.
+ `exceptions` – The **array** of **exception** objects\.

Include detailed information about the error in one or more **exception** objects\.

**`exception`**

All fields are optional except `id`\.
+ `id` – A 64\-bit identifier for the exception, unique among segments in the same trace, in **16 hexadecimal digits**\.
+ `message` – The exception message\.
+ `type` – The exception type\.
+ `remote` – **boolean** indicating that the exception was caused by an error returned by a downstream service\.
+ `truncated` – **integer** indicating the number of stack frames that are omitted from the `stack`\.
+ `skipped` – **integer** indicating the number of exceptions that were skipped between this exception and its child, that is, the exception that it caused\.
+ `cause` – Exception ID of the exception's parent, that is, the exception that caused this exception\.
+ `stack` – **array** of **stackFrame** objects\.

If available, record information about the call stack in **stackFrame** objects\.

**`stackFrame`**

All fields are optional\.
+ `path` – The relative path to the file\.
+ `line` – The line in the file\.
+ `label` – The function or method name\.

## SQL queries<a name="api-segmentdocuments-sql"></a>

You can create subsegments for queries that your application makes to an SQL database\.

**`sql`**

All fields are optional\.
+ `connection_string` – For SQL Server or other database connections that don't use URL connection strings, record the connection string, excluding passwords\.
+ `url` – For a database connection that uses a URL connection string, record the URL, excluding passwords\.
+ `sanitized_query` – The database query, with any user provided values removed or replaced by a placeholder\.
+ `database_type` – The name of the database engine\.
+ `database_version` – The version number of the database engine\.
+ `driver_version` – The name and version number of the database engine driver that your application uses\.
+ `user` – The database username\.
+ `preparation` – `call` if the query used a `PreparedCall`; `statement` if the query used a `PreparedStatement`\.

**Example Subsegment with an SQL Query**  

```
{
  "id": "3fd8634e78ca9560",
  "start_time": 1484872218.696,
  "end_time": 1484872218.697,
  "name": "ebdb@aawijb5u25wdoy.cpamxznpdoq8.us-west-2.rds.amazonaws.com",
  "namespace": "remote",
  "sql" : {
    "url": "jdbc:postgresql://aawijb5u25wdoy.cpamxznpdoq8.us-west-2.rds.amazonaws.com:5432/ebdb",
    "preparation": "statement",
    "database_type": "PostgreSQL",
    "database_version": "9.5.4",
    "driver_version": "PostgreSQL 9.4.1211.jre7",
    "user" : "dbuser",
    "sanitized_query" : "SELECT  *  FROM  customers  WHERE  customer_id=?;"
  }
}
```