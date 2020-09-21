# AWS X\-Ray sample application<a name="xray-scorekeep"></a>

The AWS X\-Ray [eb\-java\-scorekeep](https://github.com/awslabs/eb-java-scorekeep/tree/xray) sample app, available on GitHub, shows the use of the AWS X\-Ray SDK to instrument incoming HTTP calls, DynamoDB SDK clients, and HTTP clients\. The sample app uses AWS Elastic Beanstalk features to create DynamoDB tables, compile Java code on instance, and run the X\-Ray daemon without any additional configuration\.

![\[Scorekeep uses the AWS X-Ray SDK to instrument incoming HTTP calls, DynamoDB SDK clients, and HTTP clients\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-flow.png)

The sample is an instrumented version of the [Scorekeep](https://github.com/awslabs/eb-java-scorekeep) project on AWSLabs\. It includes a front\-end web app, the API that it calls, and the DynamoDB tables that it uses to store data\. All the components are hosted in an Elastic Beanstalk environment for portability and ease of deployment\.

Basic instrumentation with [filters](xray-sdk-java-filters.md), [plugins](xray-sdk-java-configuration.md), and [instrumented AWS SDK clients](xray-sdk-java-awssdkclients.md) is shown in the project's `xray-gettingstarted` branch\. This is the branch that you deploy in the [getting started tutorial](xray-gettingstarted.md)\. Because this branch only includes the basics, you can diff it against the `master` branch to quickly understand the basics\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-after-github.png)

The sample application shows basic instrumentation in these files:
+ **HTTP request filter** – [https://github.com/awslabs/eb-java-scorekeep/tree/xray-gettingstarted/src/main/java/scorekeep/WebConfig.java](https://github.com/awslabs/eb-java-scorekeep/tree/xray-gettingstarted/src/main/java/scorekeep/WebConfig.java)
+ **AWS SDK client instrumentation** – [https://github.com/awslabs/eb-java-scorekeep/tree/xray-gettingstarted/build.gradle](https://github.com/awslabs/eb-java-scorekeep/tree/xray-gettingstarted/build.gradle)

The `xray` branch of the application adds the use of [HTTPClient](xray-sdk-java-httpclients.md), [Annotations](xray-sdk-java-segment.md), [SQL queries](xray-sdk-java-sqlclients.md), [custom subsegments](xray-sdk-java-subsegments.md), an instrumented [AWS Lambda](xray-services-lambda.md) function, and [instrumented initialization code and scripts](scorekeep-startup.md)\. This service map shows the `xray` branch running without a connected SQL database:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap.png)

To support user log\-in and AWS SDK for JavaScript use in the browser, the `xray-cognito` branch adds Amazon Cognito to support user authentication and authorization\. With credentials retrieved from Amazon Cognito, the web app also sends trace data to X\-Ray to record request information from the client's point of view\. The browser client appears as its own node on the service map, and records additional information, including the URL of the page that the user is viewing, and the user's ID\.

Finally, the `xray-worker` branch adds an instrumented Python Lambda function that runs independently, processing items from an Amazon SQS queue\. Scorekeep adds an item to the queue each time a game ends\. The Lambda worker, triggered by CloudWatch Events, pulls items from the queue every few minutes and processes them to store game records in Amazon S3 for analysis\.

With all features enabled, Scorekeep's service map looks like this:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-allfeatures.png)

For instructions on using the sample application with X\-Ray, see the [getting started tutorial](xray-gettingstarted.md)\. In addition to the basic use of the X\-Ray SDK for Java discussed in the tutorial, the sample also shows how to use the following features\.

**Topics**
+ [Getting started with AWS X\-Ray through the AWS CLI](scorekeep-ubuntu.md)
+ [Manually instrumenting AWS SDK clients](scorekeep-sdkclients.md)
+ [Creating additional subsegments](scorekeep-subsegments.md)
+ [Recording annotations, metadata, and user IDs](scorekeep-annotations.md)
+ [Instrumenting outgoing HTTP calls](scorekeep-httpclient.md)
+ [Instrumenting calls to a PostgreSQL database](scorekeep-postgresql.md)
+ [Instrumenting AWS Lambda functions](scorekeep-lambda.md)
+ [Instrumenting Amazon ECS applications](scorekeep-ecs.md)
+ [Instrumenting startup code](scorekeep-startup.md)
+ [Instrumenting scripts](scorekeep-scripts.md)
+ [Instrumenting a web app client](scorekeep-client.md)
+ [Using instrumented clients in worker threads](scorekeep-workerthreads.md)
+ [Deep linking to the X\-Ray console](scorekeep-deeplinks.md)