# Cross\-account tracing<a name="xray-console-crossaccount"></a>

AWS X\-Ray supports *cross\-account observability*, enabling you to monitor and troubleshoot applications that span multiple accounts within an AWS Region\. You can seamlessly search, visualize, and analyze your metrics, logs, and traces in any of the linked accounts as if you were operating in a single account\. This provides a complete view of requests that travel across multiple accounts\. You can view cross\-account traces in the X\-Ray service map and traces pages within the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/)\.

The shared observability data can include any of these three types of telemetry: 
+ Metrics in Amazon CloudWatch
+ Log groups in Amazon CloudWatch Logs
+ Traces in AWS X\-Ray

## Configure cross\-account observability<a name="xray-console-crossaccount-configure"></a>

To turn on cross\-account observability, set up one or more AWS *monitoring* accounts and link them with multiple *source* accounts\. A monitoring account is a central AWS account that can view and interact with observability data that's generated from source accounts\. A source account is an individual AWS account that generates observability data for the resources that it contains\. Source accounts share their observability data with monitoring accounts\. Each source account can send traces to up to five monitoring accounts\. Traces sent from a source account to a single monitoring account don't incur charges, but you will be charged for traces that you send to additional monitoring accounts\. See [AWS X\-Ray pricing](https://aws.amazon.com/xray/pricing/) and [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) for more information\.

To create links between monitoring accounts and source accounts, use the CloudWatch console or the new Observability Access Manager commands in the AWS CLI and API\. For more information, see [CloudWatch cross\-account observability](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Unified-Cross-Account.html)\. 

## Viewing cross\-account traces<a name="xray-console-crossaccount-view"></a>

Cross\-account traces are displayed in the monitoring account\. Each source account displays only local traces for that specific account\. The following sections assume that you're signed in to the monitoring account and have opened the Amazon CloudWatch console\. On both the service map and traces pages, a monitoring account badge is displayed in the upper\-right corner\.

![\[Monitoring account badge\]](http://docs.aws.amazon.com/xray/latest/devguide/images/crossaccount-monitoring-account.png)

### Service map<a name="xray-console-crossaccount-view-servicemap"></a>

In the CloudWatch console, choose **Service map** under **X\-Ray traces** from the left navigation pane\. By default, the service map displays nodes for all source accounts that send traces to the monitoring account, and nodes for the monitoring account itself\. On the service map, choose **Filters** from the upper left to filter the service map using the **Accounts** drop\-down\. After an account filter is applied, service nodes from accounts that don't match the current filter are grayed out\.

![\[Filtered service map\]](http://docs.aws.amazon.com/xray/latest/devguide/images/crossaccount-servicemap-account-filter.png)

 When you choose a service node, the node details pane includes the service's account ID and label\. 

![\[Node detail pane\]](http://docs.aws.amazon.com/xray/latest/devguide/images/crossaccount-servicemap-node-detail.png)

In the upper\-right corner of the service map, choose **List view** to see a list of service nodes\. The list of service nodes includes services from the monitoring account and all configured source accounts\. Filter the list of nodes by **Account label** or **Account id** by choosing them from the **Nodes** filter\.

![\[Filtered service list\]](http://docs.aws.amazon.com/xray/latest/devguide/images/crossaccount-servicelist-account-filter.png)

### Traces<a name="xray-console-crossaccount-view-traces"></a>

View trace details for traces that span multiple accounts by opening the CloudWatch console from the monitoring account, and choosing **Traces** under **X\-Ray traces** in the left navigation pane\. You can also open this page by choosing a node in the X\-Ray **Service map**, and then choosing **View traces** from the node details pane\. 

The **Traces** page supports querying by account ID\. To get started, [enter a query](xray-console-filters.md) that includes one or more account IDs\. The following example queries for traces that have passed through account ID X* *or *Y*:

```
service(id(account.id:"X")) OR service(id(account.id:"Y"))
```

![\[Query traces by account\]](http://docs.aws.amazon.com/xray/latest/devguide/images/crossaccount-traces-query-by-account.png)

Refine your query by **Account**\. Select one or more accounts from the list, and choose **Add to query**\. 

![\[Refine trace query by account\]](http://docs.aws.amazon.com/xray/latest/devguide/images/crossaccount-traces-refine-by-account.png)

### Trace details<a name="xray-console-crossaccount-trace-details"></a>

View details for a trace by choosing it from the **Traces** list at the bottom of the **Traces** page\. The **Trace details** displays, including a trace map with service nodes from across all accounts that the trace passed through\. Choose a specific service node to see its corresponding account\. 

The **Segments timeline** section displays the account details for each segment in the timeline\.

![\[Segments timeline\]](http://docs.aws.amazon.com/xray/latest/devguide/images/crossaccount-traces-segment-timeline.png)