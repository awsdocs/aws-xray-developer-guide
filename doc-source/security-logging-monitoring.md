# Logging and monitoring in AWS X\-Ray<a name="security-logging-monitoring"></a>

Monitoring is an important part of maintaining the reliability, availability, and performance of your AWS solutions\. You should collect monitoring data from all of the parts of your AWS solution so that you can more easily debug a multi\-point failure if one occurs\. AWS provides several tools for monitoring your X\-Ray resources and responding to potential incidents:

**AWS CloudTrail Logs**  
AWS X\-Ray integrates with AWS CloudTrail to record API actions made by a user, a role, or an AWS service in X\-Ray\. You can use CloudTrail to monitor X\-Ray API requests in real time and store logs in Amazon S3, Amazon CloudWatch Logs, and Amazon CloudWatch Events\. For more information, see [Logging X\-Ray API calls with AWS CloudTrail](xray-api-cloudtrail.md)\.

**AWS Config Tracking**  
AWS X\-Ray integrates with AWS Config to record configuration changes made to your X\-Ray encryption resources\. You can use AWS Config to inventory X\-Ray encryption resources, audit the X\-Ray configuration history, and send notifications based on resource changes\. For more information, see [Tracking X\-Ray encryption configuration changes with AWS Config](xray-api-config.md)\.

**Amazon CloudWatch Monitoring**  
You can use the X\-Ray SDK for Java to publish unsampled Amazon CloudWatch metrics from your collected X\-Ray segments\. These metrics are derived from the segment’s start and end time, and the error, fault and throttled status flags\. Use these trace metrics to expose retries and dependency issues within subsegments\. For more information, see [AWS X\-Ray metrics for the X\-Ray SDK for Java](xray-sdk-java-monitoring.md)\.