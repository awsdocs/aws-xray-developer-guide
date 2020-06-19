# Integrating AWS X\-Ray with other AWS services<a name="xray-services"></a>

Other AWS services provide integration with AWS X\-Ray by adding a tracing header to requests, running the X\-Ray daemon, or making sampling decisions and uploading trace data to X\-Ray\.

**Note**  
The X\-Ray SDKs include plugins for additional integration with AWS services\. For example, you can use the X\-Ray SDK for Java's Elastic Beanstalk plugin to add information about the Elastic Beanstalk environment that runs your application including the environment name and ID\.

**Topics**
+ [Amazon API Gateway active tracing support for AWS X\-Ray](xray-services-apigateway.md)
+ [Amazon EC2 and AWS App Mesh](xray-services-appmesh.md)
+ [AWS AppSync and AWS X\-Ray](xray-services-appsync.md)
+ [Logging X\-Ray API calls with AWS CloudTrail](xray-api-cloudtrail.md)
+ [Monitoring endpoints and APIs with CloudWatch](xray-services-cloudwatch.md)
+ [Tracking X\-Ray encryption configuration changes with AWS Config](xray-api-config.md)
+ [Amazon Elastic Compute Cloud and AWS X\-Ray](xray-services-ec2.md)
+ [AWS Elastic Beanstalk and AWS X\-Ray](xray-services-beanstalk.md)
+ [Elastic Load Balancing and AWS X\-Ray](xray-services-elb.md)
+ [AWS Lambda and AWS X\-Ray](xray-services-lambda.md)
+ [Amazon SNS and AWS X\-Ray](xray-services-sns.md)
+ [Amazon SQS and AWS X\-Ray](xray-services-sqs.md)