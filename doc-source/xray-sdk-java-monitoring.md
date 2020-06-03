# AWS X\-Ray metrics for the X\-Ray SDK for Java<a name="xray-sdk-java-monitoring"></a>

This topic describes the AWS X\-Ray namespace, metrics, and dimensions\. You can use the X\-Ray SDK for Java to publish unsampled Amazon CloudWatch metrics from your collected X\-Ray segments\. These metrics are derived from the segment’s start and end time, and the error, fault, and throttled status flags\. Use these trace metrics to expose retries and dependency issues within subsegments\. 

CloudWatch is essentially a metrics repository\. A metric is the fundamental concept in CloudWatch and represents a time\-ordered set of data points\. You \(or AWS services\) publish metrics data points into CloudWatch and you retrieve statistics about those data points as an ordered set of time\-series data\. 

Metrics are uniquely defined by a name, a namespace, and one or more dimensions\. Each data point has a timestamp and, optionally, a unit of measure\. When you request statistics, the returned data stream is identified by namespace, metric name, and dimension\. 

For more information about CloudWatch, see the [https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/)\. 

## X\-Ray CloudWatch metrics<a name="xray-sdk-java-monitoring-metrics"></a>

The `ServiceMetrics/SDK` namespace includes the following metrics\.


| Metric | Statistics available | Description | Units | 
| --- | --- | --- | --- | 
|  `Latency`  |  Average, Minimum, Maximum, Count  |  The difference between the start and end time\. Average, minimum, and maximum all describe operational latency\. Count describes call count\.  |  Milliseconds  | 
|  `ErrorRate`  |  Average, Sum  |  The rate of requests that failed with a `4xx Client Error` status code, resulting in an error\.  |  Percent  | 
|  `FaultRate`  |  Average, Sum  |  The rate of traces that failed with a `5xx Server Error` status code, resulting in a fault\.  |  Percent  | 
|  `ThrottleRate`  |  Average, Sum  |  The rate of throttled traces that return a `419` status code\. This is a subset of the `ErrorRate` metric\.   |  Percent  | 
|  `OkRate`  |  Average, Sum  |  The rate of traced requests resulting in an `OK` status code\.   |  Percent  | 

## X\-Ray CloudWatch dimensions<a name="xray-sdk-java-monitoring-dimensions"></a>

Use the dimensions in the following table to refine the metrics returned for your X\-Ray instrumented Java applications\.


| Dimension | Description | 
| --- | --- | 
|  `ServiceType`  |  The type of the service, for example, `AWS::EC2::Instance` or `NONE`, if not known\.  | 
|  `ServiceName`  |  The canonical name for the service\.  | 

## Enable X\-Ray CloudWatch metrics<a name="xray-sdk-java-monitoring-enable"></a>

Use the following procedure to enable trace metrics in your instrumented Java application\.

**To configure trace metrics**

1. Add the `aws-xray-recorder-sdk-metrics` package as a Maven dependency\. For more information, see [X\-Ray SDK for Java Submodules](xray-sdk-java.md#xray-sdk-java-submodules)\.

1. Enable a new `MetricsSegmentListener()` as part of the global recorder build\.  
**Example src/com/myapp/web/Startup\.java**  

   ```
   import com.amazonaws.xray.AWSXRay;
   import com.amazonaws.xray.AWSXRayRecorderBuilder;
   import com.amazonaws.xray.plugins.EC2Plugin;
   import com.amazonaws.xray.plugins.ElasticBeanstalkPlugin;
   import com.amazonaws.xray.strategy.sampling.LocalizedSamplingStrategy;
   
   @Configuration
   public class WebConfig {
   ...
     static {
       AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder
                                           .standard()
                                           .withPlugin(new EC2Plugin())
                                           .withPlugin(new ElasticBeanstalkPlugin())
                                           .withSegmentListener(new MetricsSegmentListener());
   
       URL ruleFile = WebConfig.class.getResource("/sampling-rules.json");
       builder.withSamplingStrategy(new LocalizedSamplingStrategy(ruleFile));
   
       AWSXRay.setGlobalRecorder(builder.build());
     }
   }
   ```

1. Deploy the CloudWatch agent to collect metrics using Amazon Elastic Compute Cloud \(Amazon EC2\), Amazon Elastic Container Service \(Amazon ECS\), or Amazon Elastic Kubernetes Service \(Amazon EKS\):
   +  To configure Amazon EC2, see [Deploying the CloudWatch Agent and the X\-Ray Daemon on Amazon EC2](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy_servicelens_CloudWatch_agent_deploy_EC2.html)\.
   +  To configure Amazon ECS, see [Deploying the CloudWatch Agent and the X\-Ray Daemon on Amazon ECS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy_servicelens_CloudWatch_agent_deploy_ECS.html)\.
   +  To configure Amazon EKS, see [Deploying the CloudWatch Agent and the X\-Ray Daemon on Amazon EKS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy_servicelens_CloudWatch_agent_deploy_EKS.html)\.

1. Configure the SDK to communicate with the CloudWatch agent\. By default, the SDK communicates with the CloudWatch agent on the address `127.0.0.1`\. You can configure alternate addresses by setting the environment variable or Java property to `address:port`\.  
**Example Environment variable**  

   ```
   AWS_XRAY_METRICS_DAEMON_ADDRESS=address:port
   ```  
**Example Java property**  

   ```
   com.amazonaws.xray.metrics.daemonAddress=address:port
   ```

**To validate configuration**

1. Sign in to the AWS Management Console and open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. Open the **Metrics** tab to observe the influx of your metrics\. 

1. \(Optional\) In the CloudWatch console, on the **Logs** tab, open the `ServiceMetricsSDK` log group\. Look for a log stream that matches the host metrics, and confirm the log messages\.