# Integrating AWS X\-Ray with other AWS services<a name="xray-services"></a>

Many AWS services provide varying levels of X\-Ray integration, including sampling and adding headers to incoming requests, running the X\-Ray daemon, and automatically sending trace data to X\-Ray\. Integration with X\-Ray can include the following:
+ *Active instrumentation* – Samples and instruments incoming requests
+ *Passive instrumentation* – Instruments requests that have been sampled by another service
+ *Request tracing* – Adds a tracing header to all incoming requests and propagates it downstream
+ *Tooling* – Runs the X\-Ray daemon to receive segments from the X\-Ray SDK

**Note**  
The X\-Ray SDKs include plugins for additional integration with AWS services\. For example, you can use the X\-Ray SDK for Java Elastic Beanstalk plugin to add information about the Elastic Beanstalk environment that runs your application, including the environment name and ID\.

Here are some examples of AWS services that are integrated with X\-Ray:
+ [AWS Distro for OpenTelemetry \(ADOT\)](xray-services-adot.md) – With ADOT, engineers can instrument their applications once and send correlated metrics and traces to multiple AWS monitoring solutions including Amazon CloudWatch, AWS X\-Ray, Amazon OpenSearch Service, and Amazon Managed Service for Prometheus\.
+ [AWS Lambda](xray-services-lambda.md) – Active and passive instrumentation of incoming requests on all runtimes\. AWS Lambda adds two nodes to your service map, one for the AWS Lambda service, and one for the function\. When you enable instrumentation, AWS Lambda also runs the X\-Ray daemon on Java and Node\.js runtimes for use with the X\-Ray SDK\.
+ [Amazon API Gateway](xray-services-apigateway.md) – Active and passive instrumentation\. API Gateway uses sampling rules to determine which requests to record, and adds a node for the gateway stage to your service map\. 
+ [AWS Elastic Beanstalk](xray-services-beanstalk.md) – Tooling\. Elastic Beanstalk includes the X\-Ray daemon on the following platforms:
  + *Java SE* – 2\.3\.0 and later configurations
  + *Tomcat* – 2\.4\.0 and later configurations
  + *Node\.js* – 3\.2\.0 and later configurations
  + *Windows Server* – All configurations other than Windows Server Core that have been released after December 9th, 2016

  You can use the Elastic Beanstalk console to tell Elastic Beanstalk to run the daemon on these platforms, or use the `XRayEnabled` option in the `aws:elasticbeanstalk:xray` namespace\. 
+ [Elastic Load Balancing](xray-services-elb.md) – Request tracing on Application Load Balancers\. The Application Load Balancer adds the trace ID to the request header before sending it to a target group\.
+ [Amazon EventBridge](xray-services-eventbridge.md) – Passive instrumentation\. If a service that publishes events to EventBridge is instrumented with the X\-Ray SDK, event targets will receive the tracing header and can continue to propagate the original trace ID\. 
+ [Amazon Simple Notification Service](xray-services-sns.md) – Passive instrumentation\. If an Amazon SNS publisher traces its client with the X\-Ray SDK, subscribers can retrieve the tracing header and continue to propagate the original trace from the publisher with the same trace ID\. 
+ [Amazon Simple Queue Service](xray-services-sqs.md) – Passive instrumentation\. If a service traces requests by using the X\-Ray SDK, Amazon SQS can send the tracing header and continue to propagate the original trace from the sender to the consumer with a consistent trace ID\. 

Choose from the following topics to explore the full set of integrated AWS services\.

**Topics**
+ [AWS Distro for OpenTelemetry and AWS X\-Ray](xray-services-adot.md)
+ [Amazon API Gateway active tracing support for AWS X\-Ray](xray-services-apigateway.md)
+ [Amazon EC2 and AWS App Mesh](xray-services-appmesh.md)
+ [AWS App Runner and X\-Ray](xray-services-app-runner.md)
+ [AWS AppSync and AWS X\-Ray](xray-services-appsync.md)
+ [Logging X\-Ray API calls with AWS CloudTrail](xray-api-cloudtrail.md)
+ [Monitoring endpoints and APIs with CloudWatch](xray-services-cloudwatch.md)
+ [Tracking X\-Ray encryption configuration changes with AWS Config](xray-api-config.md)
+ [Amazon Elastic Compute Cloud and AWS X\-Ray](xray-services-ec2.md)
+ [AWS Elastic Beanstalk and AWS X\-Ray](xray-services-beanstalk.md)
+ [Elastic Load Balancing and AWS X\-Ray](xray-services-elb.md)
+ [Amazon EventBridge and AWS X\-Ray](xray-services-eventbridge.md)
+ [AWS Lambda and AWS X\-Ray](xray-services-lambda.md)
+ [Amazon SNS and AWS X\-Ray](xray-services-sns.md)
+ [AWS Step Functions and AWS X\-Ray](xray-services-stepfunctions.md)
+ [Amazon SQS and AWS X\-Ray](xray-services-sqs.md)
+ [Amazon S3 and AWS X\-Ray](xray-services-s3.md)