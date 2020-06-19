# What is AWS X\-Ray?<a name="aws-xray"></a>

AWS X\-Ray is a service that collects data about requests that your application serves, and provides tools you can use to view, filter, and gain insights into that data to identify issues and opportunities for optimization\. For any traced request to your application, you can see detailed information not only about the request and response, but also about calls that your application makes to downstream AWS resources, microservices, databases and HTTP web APIs\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-timeline.png)

The X\-Ray SDK provides: 
+ **Interceptors** to add to your code to trace incoming HTTP requests
+ **Client handlers** to instrument AWS SDK clients that your application uses to call other AWS services
+ An **HTTP client** to use to instrument calls to other internal and external HTTP web services

The SDK also supports instrumenting calls to SQL databases, automatic AWS SDK client instrumentation, and other features\.

![\[How the X-Ray SDK works\]](http://docs.aws.amazon.com/xray/latest/devguide/images/architecture-dataflow.png)

Instead of sending trace data directly to X\-Ray, the SDK sends JSON segment documents to a daemon process listening for UDP traffic\. The **[X\-Ray daemon](xray-daemon.md)** buffers segments in a queue and uploads them to X\-Ray in batches\. The daemon is available for Linux, Windows, and macOS, and is included on AWS Elastic Beanstalk and AWS Lambda platforms\.

X\-Ray uses trace data from the AWS resources that power your cloud applications to generate a detailed **service graph**\. The service graph shows the client, your front\-end service, and backend services that your front\-end service calls to process requests and persist data\. Use the service graph to identify bottlenecks, latency spikes, and other issues to solve to improve the performance of your applications\.

![\[Service graph shows the client, front-end service, and backend services that your front-end service calls to process requests and persist data\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-after-github.png)

See the [getting started tutorial](xray-gettingstarted.md) to start using X\-Ray in just a few minutes with an instrumented sample application\. Or [keep reading](xray-usage.md) to learn about the languages, frameworks, and services that work with X\-Ray\.