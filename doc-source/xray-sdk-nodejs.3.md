# AWS X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs"></a>

The X\-Ray SDK for Node\.js is a library for Express web applications and Node\.js Lambda functions that provides classes and methods for generating and sending trace data to the X\-Ray daemon\. Trace data includes information about incoming HTTP requests served by the application, and calls that the application makes to downstream services using the AWS SDK or HTTP clients\.

**Note**  
The X\-Ray SDK for Node\.js is an open source project\. You can follow the project and submit issues and pull requests on GitHub: [github\.com/aws/aws\-xray\-sdk\-node](https://github.com/aws/aws-xray-sdk-node)

If you use Express, start by [adding the SDK as middleware](xray-sdk-nodejs-middleware.md) on your application server to trace incoming requests\. The middleware creates a [segment](xray-concepts.md#xray-concepts-segments) for each traced request, and completes the segment when the response is sent\. While the segment is open you can use the SDK client's methods to add information to the segment and create subsegments to trace downstream calls\. The SDK also automatically records exceptions that your application throws while the segment is open\.

For Lambda functions called by an instrumented application or service, Lambda reads the [tracing header](xray-concepts.md#xray-concepts-tracingheader) and traces sampled requests automatically\. For other functions, you can [configure Lambda](xray-services-lambda.md) to sample and trace incoming requests\. In either case, Lambda creates the segment and provides it to the X\-Ray SDK\.

**Note**  
On Lambda, the X\-Ray SDK is optional\. If you don't use it in your function, your service map will still include a node for the Lambda service, and one for each Lambda function\. By adding the SDK, you can instrument your function code to add subsegments to the function segment recorded by Lambda\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

Next, use the X\-Ray SDK for Node\.js to [instrument your AWS SDK for JavaScript in Node\.js clients](xray-sdk-nodejs-awssdkclients.md)\. Whenever you make a call to a downstream AWS service or resource with an instrumented client, the SDK records information about the call in a subsegment\. AWS services and the resources that you access within the services appear as downstream nodes on the service map to help you identify errors and throttling issues on individual connections\.

The X\-Ray SDK for Node\.js also provides instrumentation for downstream calls to HTTP web APIs and SQL queries\. [Wrap your HTTP client in the SDK's capture method](xray-sdk-nodejs-httpclients.md) to record information about outgoing HTTP calls\. For SQL clients, [use the capture method for your database type](xray-sdk-nodejs-sqlclients.md)\.

The middleware applies sampling rules to incoming requests to determine which requests to trace\. You can [configure the X\-Ray SDK for Node\.js](xray-sdk-nodejs-configuration.md) to adjust the sampling behavior or to record information about the AWS compute resources on which your application runs\.

Record additional information about requests and the work that your application does in [annotations and metadata](xray-sdk-nodejs-segment.md)\. Annotations are simple key\-value pairs that are indexed for use with [filter expressions](xray-console-filters.md), so that you can search for traces that contain specific data\. Metadata entries are less restrictive and can record entire objects and arrays — anything that can be serialized into JSON\.

**Annotations and Metadata**  
Annotations and metadata are arbitrary text that you add to segments with the X\-Ray SDK\. Annotations are indexed for use with filter expressions\. Metadata are not indexed, but can be viewed in the raw segment with the X\-Ray console or API\. Anyone that you grant read access to X\-Ray can view this data\.

When you have a lot of instrumented clients in your code, a single request segment can contain a large number of subsegments, one for each call made with an instrumented client\. You can organize and group subsegments by wrapping client calls in [custom subsegments](xray-sdk-nodejs-subsegments.md)\. You can create a custom subsegment for an entire function or any section of code, and record metadata and annotations on the subsegment instead of writing everything on the parent segment\.

For reference documentation about the SDK's classes and methods, see the [AWS X\-Ray SDK for Node\.js API Reference](https://docs.aws.amazon.com//xray-sdk-for-nodejs/latest/reference)\.

## Requirements<a name="xray-sdk-nodejs-requirements"></a>

The X\-Ray SDK for Node\.js requires Node\.js and the following libraries:
+ `atomic-batcher` – 1\.0\.2
+ `cls-hooked` – 4\.2\.2
+ `pkginfo` – 0\.4\.0
+ `semver` – 5\.3\.0

The SDK pulls these libraries in when you install it with NPM\.

To trace AWS SDK clients, the X\-Ray SDK for Node\.js requires a minimum version of the AWS SDK for JavaScript in Node\.js\.
+ `aws-sdk` – 2\.7\.15

## Dependency management<a name="xray-sdk-nodejs-dependencies"></a>

The X\-Ray SDK for Node\.js is available from NPM\.
+ **Package** – [https://www.npmjs.com/package/aws-xray-sdk](https://www.npmjs.com/package/aws-xray-sdk)

For local development, install the SDK in your project directory with npm\.

```
~/nodejs-xray$ npm install aws-xray-sdk
aws-xray-sdk@3.0.0
├─┬ aws-xray-sdk-core@3.0.0
│ ├── atomic-batcher@1.0.2
│ ├─┬ cls-hooked@4.2.2
│ │ ├─┬ async-hook-jl@1.7.6
│ │ │ └── stack-chain@1.3.7
│ │ ├── emitter-listener@1.1.2
│ │ └── shimmer@1.2.1
│ ├── pkginfo@0.4.1 
│ └── semver@5.7.1
├── aws-xray-sdk-express@3.0.0
├── aws-xray-sdk-mysql@3.0.0
└── aws-xray-sdk-postgres@3.0.0
```

Use the `--save` option to save the SDK as a dependency in your application's `package.json`\.

```
~/nodejs-xray$ npm install aws-xray-sdk --save
aws-xray-sdk@3.0.0
```

## Node\.js samples<a name="xray-sdk-nodejs-sample"></a>

Work with the AWS X\-Ray SDK for Node\.js to get an end\-to\-end view of requests as they travel through your Node\.js applications\. 
+ [Node\.js sample application](https://github.com/aws-samples/aws-xray-sdk-node-sample) on GitHub\.