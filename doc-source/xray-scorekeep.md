# AWS X\-Ray sample application<a name="xray-scorekeep"></a>

The AWS X\-Ray [eb\-java\-scorekeep](https://github.com/awslabs/eb-java-scorekeep/tree/xray) sample app, available on GitHub, shows the use of the AWS X\-Ray SDK to instrument incoming HTTP calls, DynamoDB SDK clients, and HTTP clients\. The sample app uses AWS CloudFormation to create DynamoDB tables, compile Java code on instance, and run the X\-Ray daemon without any additional configuration\.

See the [Scorekeep tutorial](scorekeep-tutorial.md) to start installing and using an instrumented sample application, using the AWS console or the AWS CLI\.

![\[Scorekeep uses the AWS X-Ray SDK to instrument incoming HTTP calls, DynamoDB SDK clients, and HTTP clients\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-flow.png)

The sample includes a front\-end web app, the API that it calls, and the DynamoDB tables that it uses to store data\. Basic instrumentation with [filters](xray-sdk-java-filters.md), [plugins](xray-sdk-java-configuration.md), and [instrumented AWS SDK clients](xray-sdk-java-awssdkclients.md) is shown in the project's `xray-gettingstarted` branch\. This is the branch that you deploy in the [getting started tutorial](scorekeep-tutorial.md)\. Because this branch only includes the basics, you can diff it against the `master` branch to quickly understand the basics\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-before-ECS.png)

The sample application shows basic instrumentation in these files:
+ **HTTP request filter** – [https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/WebConfig.java](https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/WebConfig.java)
+ **AWS SDK client instrumentation** – [https://github.com/awslabs/eb-java-scorekeep/tree/xray/build.gradle](https://github.com/awslabs/eb-java-scorekeep/tree/xray/build.gradle)

The `xray` branch of the application includes the use of [HTTPClient](xray-sdk-java-httpclients.md), [Annotations](xray-sdk-java-segment.md), [SQL queries](xray-sdk-java-sqlclients.md), [custom subsegments](xray-sdk-java-subsegments.md), an instrumented [AWS Lambda](xray-services-lambda.md) function, and [instrumented initialization code and scripts](scorekeep-startup.md)\.

To support user log\-in and AWS SDK for JavaScript use in the browser, the `xray-cognito` branch adds Amazon Cognito to support user authentication and authorization\. With credentials retrieved from Amazon Cognito, the web app also sends trace data to X\-Ray to record request information from the client's point of view\. The browser client appears as its own node on the service map, and records additional information, including the URL of the page that the user is viewing, and the user's ID\.

Finally, the `xray-worker` branch adds an instrumented Python Lambda function that runs independently, processing items from an Amazon SQS queue\. Scorekeep adds an item to the queue each time a game ends\. The Lambda worker, triggered by CloudWatch Events, pulls items from the queue every few minutes and processes them to store game records in Amazon S3 for analysis\.

**Topics**
+ [Getting started with the Scorekeep sample application](scorekeep-tutorial.md)
+ [Manually instrumenting AWS SDK clients](scorekeep-sdkclients.md)
+ [Creating additional subsegments](scorekeep-subsegments.md)
+ [Recording annotations, metadata, and user IDs](scorekeep-annotations.md)
+ [Instrumenting outgoing HTTP calls](scorekeep-httpclient.md)
+ [Instrumenting calls to a PostgreSQL database](scorekeep-postgresql.md)
+ [Instrumenting AWS Lambda functions](scorekeep-lambda.md)
+ [Instrumenting startup code](scorekeep-startup.md)
+ [Instrumenting scripts](scorekeep-scripts.md)
+ [Instrumenting a web app client](scorekeep-client.md)
+ [Using instrumented clients in worker threads](scorekeep-workerthreads.md)