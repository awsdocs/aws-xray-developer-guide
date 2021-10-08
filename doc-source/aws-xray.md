# What is AWS X\-Ray?<a name="aws-xray"></a>

AWS X\-Ray is a service that collects data about requests that your application serves, and provides tools that you can use to view, filter, and gain insights into that data to identify issues and opportunities for optimization\. For any traced request to your application, you can see detailed information not only about the request and response, but also about calls that your application makes to downstream AWS resources, microservices, databases, and web APIs\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-timeline.png)

AWS X\-Ray receives traces from your application, in additionto AWS services your application uses that are already integrated with X\-Ray\. Instrumenting your application involves sending trace data for incoming and outbound requests and other events within your application, along with metadata about each request\. Many instrumentation scenarios require only configuration changes\. For example, you can instrument all incoming HTTP requests and downstream calls to AWS services that your Java application makes\. There are several SDKs, agents, and tools that can be used to instrument your application for X\-Ray tracing\.  See [Instrumenting your application](xray-instrumenting-your-app.md) for more information\. 

AWS services that are [integrated with X\-Ray](xray-services.md) can add tracing headers to incoming requests, send trace data to X\-Ray, or run the X\-Ray daemon\. For example, AWS Lambda can send trace data about requests to your Lambda functions, and run the X\-Ray daemon on workers to make it simpler to use the X\-Ray SDK\.

![\[How the X-Ray SDK works\]](http://docs.aws.amazon.com/xray/latest/devguide/images/architecture-dataflow.png)

Instead of sending trace data directly to X\-Ray, each client SDK sends JSON segment documents to a daemon process listening for UDP traffic\. The [X\-Ray daemon](xray-daemon.md) buffers segments in a queue and uploads them to X\-Ray in batches\. The daemon is available for Linux, Windows, and macOS, and is included on AWS Elastic Beanstalk and AWS Lambda platforms\.

X\-Ray uses trace data from the AWS resources that power your cloud applications to generate a detailed *service graph*\. The service graph shows the client, your front\-end service, and backend services that your front\-end service calls to process requests and persist data\. Use the service graph to identify bottlenecks, latency spikes, and other issues to solve to improve the performance of your applications\.

![\[Service graph shows the client, front-end service, and backend services that your front-end service calls to process requests and persist data\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-after-github.png)

See the [getting started tutorial](xray-gettingstarted.md) to start using X\-Ray with an instrumented sample application\.