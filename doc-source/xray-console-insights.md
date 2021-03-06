# Using insights in the AWS X\-Ray console<a name="xray-console-insights"></a>

AWS X\-Ray continuously analyzes trace data in your account to identify emergent issues in your applications\. When fault rates exceed the expected range, it creates an *insight* that records the incident and tracks its impact until it's resolved\. You can use insights to identify user impact for ongoing issues or to analyze transient issues that occurred in the past\.

The X\-Ray console also identifies nodes with ongoing incidents in the service map\. To see a summary of the insight, choose the affected node\.

![\[Service map node with insight summary.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-insights-servicemap.png)

X\-Ray creates an insight when it detects an *anomaly* in one or more nodes of the service map\. The service uses statistical modeling to predict the expected fault rates of services in your application\. In the preceding example, the anomaly is an increase in faults from AWS Elastic Beanstalk\. The Elastic Beanstalk server experienced multiple API call timeouts, causing an anomaly in the downstream nodes\.

## Enable insights in the X\-Ray console<a name="xray-console-enable-insights"></a>

Insights must be enabled for each group you want to use insights features with\. You can enable insights from the **Groups** page

1. Open the [X\-Ray console](https://console.aws.amazon.com/xray/home#/service-map)\.

1. Select an existing group or create a new one by choosing **Create group**, and then select **Enable Insights**\. For more information about configuring groups in the X\-Ray console, see [Configuring groups in the X\-Ray console](xray-console-groups.md)\.

1. In the navigation pane on the left, choose **Insights**, and then choose an insight to view\.  
![\[List of insights in the X-Ray console.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-insights.png)

**Note**  
X\-Ray uses GetInsightSummaries, GetInsight, GetInsightEvents, and GetInsightImpactGraph APIs to retrieve data from insights\. To view insights, use the AWSXrayReadOnlyAccess IAM managed policy or add the following custom policy to your IAM role:   

```
          {
             "Version": "2012-10-17",
             "Statement": [
                 {
                     "Effect": "Allow",
                     "Action": [
                         "xray:GetInsightSummaries",
                         "xray:GetInsight",
                         "xray:GetInsightEvents",
                         "xray:GetInsightImpactGraph"
                     ],
                     "Resource": [
                         "*"
                     ]
                 }
             ]
         }
```
For more information, see [How AWS X\-Ray works with IAM](security_iam_service-with-iam.md)\. 

## Enable insights notifications<a name="xray-console-insight-notifications"></a>

With insights notifications, a notification is created for each insight event, such as when an insight is created, changes significantly, or is closed\. Customers can receive these notifications through Amazon EventBridge events, and use conditional rules to take actions such as SNS notification, Lambda invocation, posting messages to an SQS queue, or any of the targets EventBridge supports\. Insights notifications are emitted on a best\-effort basis but are not guaranteed\. For more information about targets, see [Amazon EventBridge Targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-targets.html)\.

You can enable insights notifications for any insights enabled group from the **Groups** page\.

**To enable notifications for an X\-Ray group**

1. Open the [X\-Ray console](https://console.aws.amazon.com/xray/home#/service-map)\.

1. Select an existing group or create a new one by choosing **Create group**, ensure **Enable Insights** is selected, and then select **Enable Notifications**\. For more information about configuring groups in the X\-Ray console, see [Configuring groups in the X\-Ray console](xray-console-groups.md)\.

**To configure Amazon EventBridge conditional rules**

1. Open the [Amazon EventBridge console](https://console.aws.amazon.com/events/home)\.

1. Navigate to **Rules** in the left navigation bar, and choose **Create rule**\.

1. Provide a name and description for the rule\.

1. Choose **Event pattern**, and then choose **Custom pattern**\. Provide a pattern containing `"source": [ "aws.xray" ]` and `"detail-type": [ "AWS X-Ray Insight Update" ]`\. The following are some examples of possible patterns\.
   + Event pattern to match all incoming events from X\-Ray insights:

     ```
     {
     "source": [ "aws.xray" ],
     "detail-type": [ "AWS X-Ray Insight Update" ]
     }
     ```
   + Event pattern to match a specified **state** and **category**:

     ```
              
     {
     "source": [ "aws.xray" ],
     "detail-type": [ "AWS X-Ray Insight Update" ],
     "detail": {
             "State": [ "ACTIVE" ],
             "Category": [ "FAULT" ]
       }
     }
     ```

1. Select and configure the targets that you would like to invoke when an event matches this rule\.

1. \(Optional\) Provide tags to more easily identify and select this rule\.

1. Choose **Create**\.

**Note**  
 X\-Ray insights notifications sends events to Amazon EventBridge, which does not currently support customer managed CMKs\. For more information, see [Data protection in AWS X\-Ray](xray-console-encryption.md)\.

## Insight overview<a name="xray-console-insights-overview"></a><a name="anomalous-service"></a>

The overview page for an insight attempts to answer three key questions: 
+ What is the underlying issue?
+ What is the root cause?
+ What is the client impact?

The **Anomalous services** section shows a timeline for each service that illustrates the change in fault rates during the incident\. The timeline shows the number of traces with faults overlayed on a solid band that indicates the expected number of faults based on the amount of traffic recorded\. The duration of the insight is visualized by the *Incident window*\. The incident window begins when X\-Ray observes the metric becoming anomalous and persists while the insight is active\.

The following example shows an increase in faults that caused an incident:

![\[Overview page of an X-Ray insight.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-insights-overview.png)<a name="root-cause"></a>

The **Root cause** section shows a service map focused on the root cause service and the impacted path\. You may hide the unaffected nodes by selecting the eye icon in the top right of the Root cause map\. The root cause service is the farthest downstream node where X\-Ray identified an anomaly\. It can represent a service that you instrumented or an external service that your service called with an instrumented client\. For example, if you call Amazon DynamoDB with an instrumented AWS SDK client, an increase in faults from DynamoDB results in an insight with DynamoDB as the root cause\. 

To dive deeper into the root cause, select **View root cause details** on the root cause graph\. You can use the **Analytics** page to investigate the root cause and related messages\. For more information, see [Interacting with the AWS X\-Ray Analytics console](xray-console-analytics.md)\.

![\[Overview page of an X-Ray insight.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-insights-root-cause.png)<a name="impact"></a>

Faults that continue upstream in the map can impact multiple nodes and cause multiple anomalies\. If a fault is passed all the way back to the user that made the request, the result is a *client fault*\. This is a fault in the root node of the service map\. The **Impact** graph provides a timeline of the client experience for the entire group\. This experience is calculated based on percentages of the following states: **Fault**, **Error**, **Throttle**, and **Okay**\.

![\[Impact graph for an X-Ray incident.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-insights-impact.png)

This example shows an increase in traces with a fault at the root node during the time of an incident\. Incidents in downstream services don't always correspond to an increase in client errors\.

Choosing **Analyze insight** opens the X\-Ray Analytics console in a window where you can dive deep into the set of traces causing the insight\. For more information, see [Interacting with the AWS X\-Ray Analytics console](xray-console-analytics.md)\. 

## Review an insight's progress<a name="xray-console-insights-inspect"></a>

X\-Ray reevaluates insights periodically until they are resolved, and records each notable intermediate change as an event\. You can review incident events in the **Impact Timeline** on the **Inspect** page\. By default the timeline displays the most impacted service until you choose a different service\.

![\[Inspect page with impact timeline.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-insights-inspect.png)<a name="impact-analysis"></a>

To see a service map and graphs for an event, choose it from the impact timeline\. The service map shows services in your application that are affected by the incident\. Under **Impact analysis**, graphs show fault timelines for the selected node and for clients in the group\.

![\[Impact analysis graph for an X-Ray insight.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-insights-inspect-analysis.png)

To take a deeper look at the traces involved in an incident, choose **Analyze event** on the **Inspect** page\. You can use the **Analytics** page to refine the list of traces and identify affected users\. For more information, see [Interacting with the AWS X\-Ray Analytics console](xray-console-analytics.md)\.