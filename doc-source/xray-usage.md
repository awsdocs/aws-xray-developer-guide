# AWS X\-Ray use cases and requirements<a name="xray-usage"></a>

You can use the X\-Ray SDK and AWS service integration to instrument requests to your applications that are running locally or on AWS compute services such as Amazon EC2, Elastic Beanstalk, Amazon ECS and AWS Lambda\.

To instrument your application code, you use the **X\-Ray SDK**\. The SDK records data about incoming and outgoing requests and sends it to the X\-Ray daemon, which relays the data in batches to X\-Ray\. For example, when your application calls DynamoDB to retrieve user information from a DynamoDB table, the X\-Ray SDK records data from both the client request and the downstream call to DynamoDB\.

![\[X-Ray SDK records data from both the client request and downstream call to DynamoDB\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-ddb-edge.png)

Other AWS services make it easier to instrument your application's components by integrating with X\-Ray\. **Service integration** can include adding tracing headers to incoming requests, sending trace data to X\-Ray, or running the X\-Ray daemon\. For example, AWS Lambda can send trace data about requests to your Lambda functions, and run the X\-Ray daemon on workers to make it easier to use the X\-Ray SDK\.

![\[Lambda integration with the X-Ray SDK\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-lambda-node.png)

Many instrumentation scenarios require only configuration changes\. For example, you can instrument all incoming HTTP requests and downstream calls to AWS services that your Java application makes\. To do this, you add the X\-Ray SDK for Java's filter to your servlet configuration, and take the AWS SDK for Java Instrumentor submodule as a build dependency\. For advanced instrumentation, you can modify your application code to customize and annotate the data that the SDK sends to X\-Ray\.

**Topics**
+ [Supported languages and frameworks](#xray-usage-languages)
+ [Supported AWS services](#xray-usage-services)
+ [Code and configuration changes](#xray-usage-codechanges)

## Supported languages and frameworks<a name="xray-usage-languages"></a>

AWS X\-Ray provides tools and integration to support a variety of languages, frameworks, and platforms\.

**C\#**

On Windows Server, you can use the X\-Ray SDK for \.NET to instrument incoming requests, AWS SDK clients, SQL clients, and HTTP clients\. On AWS Lambda, you can use the Lambda X\-Ray integration to instrument incoming requests\.

See [AWS X\-Ray SDK for \.NET](xray-sdk-dotnet.md) for more information\.
+ **\.NET on Windows Server** – [Add a message handler](xray-sdk-dotnet-messagehandler.md#xray-sdk-dotnet-messagehandler-globalasax) to your HTTP configuration to instrument incoming requests\.
+ **C\# \.NET Core on AWS Lambda** – Enable X\-Ray on your Lambda function configuration to instrument incoming requests\.

**Go**

In any Go application, you can use the X\-Ray SDK for Go classes to instrument incoming requests, AWS SDK clients, SQL clients, and HTTP clients\. Automatic request instrumentation is available for applications that use HTTP handlers\.

On AWS Lambda, you can use the Lambda X\-Ray integration to instrument incoming requests\. Add the X\-Ray SDK for Go to your function for full instrumentation\.

See [AWS X\-Ray SDK for Go](xray-sdk-go.md) for more information\.
+ **Go web applications** – Use the [X\-Ray SDK for Go HTTP handler](xray-sdk-go-handler.md) to process incoming requests on your routes\.
+ **Go on AWS Lambda** – Enable X\-Ray on your Lambda function configuration to instrument incoming requests\. Add the X\-Ray SDK for Go to instrument AWS SDK, HTTP, and SQL clients\.

**Java**

In any Java application, you can use the X\-Ray SDK for Java classes to instrument incoming requests, AWS SDK clients, SQL clients, and HTTP clients\. Automatic request instrumentation is available for frameworks that support Java servlets\. Automatic SDK instrumentation is available through the Instrumentor submodule\.

On AWS Lambda, you can use the Lambda X\-Ray integration to instrument incoming requests\. Add the X\-Ray SDK for Java to your function for full instrumentation\.

See [AWS X\-Ray SDK for Java](xray-sdk-java.md) for more information\.
+ **Tomcat** – [Add a servlet filter](xray-sdk-java-filters.md#xray-sdk-java-filters-tomcat) to your deployment descriptor \(`web.xml`\) to instrument incoming requests\.
+ **Spring Boot** – [Add a servlet filter](xray-sdk-java-filters.md#xray-sdk-java-filters-spring) to your `WebConfig` class to instrument incoming requests\.
+ **Java on AWS Lambda** – Enable X\-Ray on your Lambda function to instrument incoming requests\. Add the X\-Ray SDK for Java to instrument AWS SDK, HTTP, and SQL clients\.
+ **Other frameworks** – Add a servlet filter if your framework supports servlets, or manually create a segment for each incoming request\.

**Node\.js**

In any Node\.js application, you can use the X\-Ray SDK for Node\.js classes to instrument incoming requests, AWS SDK clients, SQL clients, and HTTP clients\. Automatic request instrumentation is available for applications that use the Express and Restify frameworks\.

On AWS Lambda, you can use the Lambda X\-Ray integration to instrument incoming requests\. Add the X\-Ray SDK for Node\.js to your function for full instrumentation\.

See [AWS X\-Ray SDK for Node\.js](xray-sdk-nodejs.md) for more information\.
+ **Express or Restify** – [Use the X\-Ray SDK for Node\.js middleware](xray-sdk-nodejs-middleware.md) to instrument incoming requests\.
+ **Node\.js on AWS Lambda** – Enable X\-Ray on your Lambda function to instrument incoming requests\. Add the X\-Ray SDK for Node\.js to instrument AWS SDK, HTTP, and SQL clients
+ **Other frameworks** – Manually create a segment for each incoming request\.

**Python**

In any Python application, you can use the X\-Ray SDK for Python classes to instrument incoming requests, AWS SDK clients, SQL clients, and HTTP clients\. Automatic request instrumentation is available for applications that use the Django and Flask frameworks\.

On AWS Lambda, you can use the Lambda X\-Ray integration to instrument incoming requests\. Add the X\-Ray SDK for Python to your function for full instrumentation\.

See [AWS X\-Ray SDK for Python](xray-sdk-python.md) for more information\.
+ **Django or Flask** – [Use the X\-Ray SDK for Python middleware](xray-sdk-python-middleware.md) to instrument incoming requests\.
+ **Python on AWS Lambda** – Enable X\-Ray on your Lambda function configuration to instrument incoming requests\. Add the X\-Ray SDK for Python to instrument AWS SDK, HTTP, and SQL clients\.
+ **Other frameworks** – Manually create a segment for each incoming request\.

**Ruby**

In any Ruby application, you can use the X\-Ray SDK for Ruby classes to instrument incoming requests, AWS SDK clients, SQL clients, and HTTP clients\. Automatic request instrumentation is available for applications that use the Rails framework\.
+ **Rails** – Add the X\-Ray SDK for Ruby gem and railtie to your gemfile, and [configure the recorder](xray-sdk-ruby-middleware.md) in an initializer to instrument incoming requests\.
+ **Other frameworks** – [Manually create a segment](xray-sdk-ruby-middleware.md#xray-sdk-ruby-middleware-manual) for each incoming request\.

If the X\-Ray SDK isn't available for your language or platform, you can generate trace data manually and send it to the X\-Ray daemon, or directly to [the X\-Ray API](xray-api.md)\.

## Supported AWS services<a name="xray-usage-services"></a>

Several AWS services provide **X\-Ray integration**\. [Integrated services](xray-services.md) offer varying levels of integration that can include sampling and adding headers to incoming requests, running the X\-Ray daemon, and automatically sending trace data to X\-Ray\.
+ **Active instrumentation** – Samples and instruments incoming requests\.
+ **Passive instrumentation** – Instruments requests that have been sampled by another service\.
+ **Request tracing** – Adds a tracing header to all incoming requests and propagates it downstream\.
+ **Tooling** – Runs the X\-Ray daemon to receive segments from the X\-Ray SDK\. 

The following services provide X\-Ray integration:
+ **AWS Lambda** – Active and passive instrumentation of incoming requests on all runtimes\. AWS Lambda adds two nodes to your service map, one for the AWS Lambda service, and one for the function\. When you enable instrumentation, AWS Lambda also runs the X\-Ray daemon on Java and Node\.js runtimes for use with the X\-Ray SDK\. [Learn more](xray-services-lambda.md)\.
+ **Amazon API Gateway** – Active and passive instrumentation\. API Gateway uses sampling rules to determine which requests to record, and adds a node for the gateway stage to your service map\. [Learn more](xray-services-apigateway.md)\.
+ **Elastic Load Balancing** – Request tracing on application load balancers\. The application load balancer adds the trace ID to the request header before sending it to a target group\. [Learn more](xray-services-elb.md)\.
+ **AWS Elastic Beanstalk** – Tooling\. Elastic Beanstalk includes the X\-Ray daemon on the following platforms:
  + **Java SE** – 2\.3\.0 and later configurations
  + **Tomcat** – 2\.4\.0 and later configurations
  + **Node\.js** – 3\.2\.0 and later configurations
  + **Windows Server** – All configurations other than Windows Server Core that have been released since December 9th, 2016\.

  You can use the Elastic Beanstalk console to tell Elastic Beanstalk to run the daemon on these platforms, or use the `XRayEnabled` option in the `aws:elasticbeanstalk:xray` namespace\. [Learn more](xray-services-beanstalk.md)\.
+ **Amazon Simple Notification Service** – Passive instrumentation\. If an Amazon SNS publisher traces its client with the X\-Ray SDK, subscribers can retrieve the tracing header and continue to propagate the original trace from the publisher with the same trace ID\. [Learn more](xray-services-sns.md)\.
+ **Amazon Simple Queue Service** – Passive instrumentation\. If a service traces requests by using the X\-Ray SDK, Amazon SQS can send the tracing header and continue to propagate the original trace from the sender to the consumer with a consistent trace ID\. [Learn more](xray-services-sqs.md)\.

## Code and configuration changes<a name="xray-usage-codechanges"></a>

A large amount of tracing data can be generated without any functional changes to your code\. Detailed tracing of front\-end and downstream calls requires only minimal changes to build and deploy\-time configuration\.

**Examples of Code and Configuration Changes**
+ **AWS resource configuration** – Change AWS resource settings to instrument requests to a Lambda function\. Run the X\-Ray daemon on the instances in your Elastic Beanstalk environment by changing an option setting\.
+ **Build configuration** – Take X\-Ray SDK for Java submodules as a compile\-time dependency to instrument all downstream requests to AWS services, and to resources such as Amazon DynamoDB tables, Amazon SQS queues, and Amazon S3 buckets\.
+ **Application configuration** – To instrument incoming HTTP requests, add a servlet filter to your Java application, or use the X\-Ray SDK for Node\.js as middleware on your Express application\. Change sampling rules and enable plugins to instrument the Amazon EC2, Amazon ECS, and AWS Elastic Beanstalk resources that run your application\.
+ **Class or object configuration** – To instrument outgoing HTTP calls in Java, import the X\-Ray SDK for Java version of `HttpClientBuilder` instead of the Apache\.org version\.
+ **Functional changes** – Add a request handler to an AWS SDK client to instrument calls that it makes to AWS services\. Create subsegments to group downstream calls, and add debug information to segments with annotations and metadata\.