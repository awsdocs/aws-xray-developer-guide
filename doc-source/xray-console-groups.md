# Configuring groups in the X\-Ray console<a name="xray-console-groups"></a>

Groups are a collection of traces that are defined by a filter expression\. You can use groups to generate additional service graphs and supply Amazon CloudWatch metrics\. You can use the AWS X\-Ray console or X\-Ray API to create and manage groups for your services\. This topic describes how to create and manage groups by using the X\-Ray console\. For information about how to manage groups by using the X\-Ray API, see [Groups](xray-api-configuration.md#xray-api-configuration-groups)\.

You can create groups of traces for service maps, traces, or analytics\. When you create a group, the group becomes available as a filter on the group dropdown menu on all three pages: **Service map**, **Traces**, and **Analytics**\.

![\[Group menu\]](http://docs.aws.amazon.com/xray/latest/devguide/images/group-menu.png)

Groups are identified by their name or an Amazon Resource Name \(ARN\), and contain a filter expression\. The service compares incoming traces to the expression and stores them accordingly\. For more information about how to build a filter expression, see [Using filter expressions to search for traces in the console](xray-console-filters.md)\.

Updating a group's filter expression doesn't change data that's already recorded\. The update applies only to subsequent traces\. This can result in a merged graph of the new and old expressions\. To avoid this, delete a current group and create a new one\.

**Note**  
Groups are billed by the number of retrieved traces that match the filter expression\. For more information, see [AWS X\-Ray pricing](https://aws.amazon.com/xray/pricing/)\. 

**Topics**
+ [Create a group in the X\-Ray console](#xray-console-group-create-console)
+ [Apply a group](#xray-console-group-apply)
+ [Edit a group](#xray-console-group-edit)
+ [Clone a group](#xray-console-group-clone)
+ [Delete a group](#xray-console-group-delete)
+ [View group metrics in Amazon CloudWatch](#xray-console-group-cloudwatch)

## Create a group in the X\-Ray console<a name="xray-console-group-create-console"></a>

1. Sign in to the AWS Management Console and open the X\-Ray console at [https://console\.aws\.amazon\.com/xray/home](https://console.aws.amazon.com/xray/home)\.

1. Open the **Create group** page from the **Groups** page in the left navigation pane, or from the group menu on one of the following pages\.
   + **Service map**
   + **Traces**
   + **Analytics**

1. Choose **Create group** on the group menu, or on the **Groups** page\.

1. On the **Create group** page, enter a name for the group\. A group name can have a maximum of 32 characters, and contain alphanumeric characters and dashes\. Group names are case sensitive\.  
![\[Create group page\]](http://docs.aws.amazon.com/xray/latest/devguide/images/group-create.png)

1. Enter a filter expression\. For more information about how to build a filter expression, see [Using filter expressions to search for traces in the console](xray-console-filters.md)\. In the following example, the group filters for fault traces from the service `api.example.com`\. and requests to the service where the response time was greater than or equal to five seconds\.

   ```
   fault = true AND http.url CONTAINS "example/game" AND responsetime >= 5
   ```

1. In **Insights**, enable or disable insights access for the group\. For more information about insights, see [Using insights in the AWS X\-Ray console](xray-console-insights.md)\.

1. In **Tags**, enter a tag key, and optionally, a tag value\. As you add a tag, a new line appears for you to enter another tag\. Tag keys must be unique\. To delete a tag, choose **X** at the end of the tag's row\. For more information about tags, see [Tagging X\-Ray sampling rules and groups](xray-tagging.md)\.

1. Choose **Create group**\.

## Apply a group<a name="xray-console-group-apply"></a>

1. Sign in to the AWS Management Console and open the X\-Ray console at [https://console\.aws\.amazon\.com/xray/home](https://console.aws.amazon.com/xray/home)\.

1. Open one of the following pages from the navigation pane\.
   + **Service map**
   + **Traces**
   + **Analytics**

1. Do the following, in any order\.

   1. On the group menu, choose the group that you created in [Create a group in the X\-Ray console](#xray-console-group-create-console)\. The data shown on the page changes to match the filter expression set in the group\.

   1. Optionally, choose the time period for which you want to view data\. The default is **Last 5 minutes** on the **Relative** tab\.  
![\[Relative time span control\]](http://docs.aws.amazon.com/xray/latest/devguide/images/time-period.png)

      To set a specific date and time range, choose **Custom \(UTC\)**\. Times specified on this tab are in 24\-hour format\. For example, 6:00 PM is `18:00`\. Your custom time range can be a maximum of six hours\.  
![\[Custom (UTC) time span control\]](http://docs.aws.amazon.com/xray/latest/devguide/images/time-period-custom.png)

## Edit a group<a name="xray-console-group-edit"></a>

1. Sign in to the AWS Management Console and open the X\-Ray console at [https://console\.aws\.amazon\.com/xray/home](https://console.aws.amazon.com/xray/home)\.

1. Do one of the following to open the **Edit group** page\.

   1. On the **Groups** page, choose the name of a group to edit it\.

   1. On the group menu on one of the following pages, point to a group, and then choose **Edit**\.
      + **Service map**
      + **Traces**
      + **Analytics**  
![\[Edit group page\]](http://docs.aws.amazon.com/xray/latest/devguide/images/group-edit.png)

1. Although you can't rename a group, you can update the filter expression\. For more information about how to build a filter expression, see [Using filter expressions to search for traces in the console](xray-console-filters.md)\. In the following example, the group filters for fault traces from the service `api.example.com`, where the request URL address contains `example/game`, and response time for requests was greater than or equal to five seconds\.

   ```
   fault = true AND http.url CONTAINS "example/game" AND responsetime >= 5
   ```

1. In **Insights**, enable or disable insights access for the group\. For more information about insights, see [Using insights in the AWS X\-Ray console](xray-console-insights.md)\.

1. In **Tags**, edit tag keys and values\. Tag keys must be unique\. Tag values are optional; you can delete values if you want\. To delete a tag, choose **X** at the end of the tag's row\. For more information about tags, see [Tagging X\-Ray sampling rules and groups](xray-tagging.md)\.

1. When you're finished updating the filter expression, choose **Update group**\.

## Clone a group<a name="xray-console-group-clone"></a>

Cloning a group creates a new group that has the filter expression and tags of an existing group\. When you clone a group, the new group has the same name as the group from which it's cloned, with `-clone` appended to the name\.

1. Sign in to the AWS Management Console and open the X\-Ray console at [https://console\.aws\.amazon\.com/xray/home](https://console.aws.amazon.com/xray/home)\.

1. On the **Groups** page, select a group by selecting its check box\. You can only select one group at a time\.

1. On the **Actions** menu, choose **Clone group**\.  
![\[Actions menu, Clone group option\]](http://docs.aws.amazon.com/xray/latest/devguide/images/group-clone.png)

1. On the **Create group** page, the name of the group is *group\-name*`-clone`\. Optionally, enter a new name for the group\. A group name can have a maximum of 32 characters, and contain alphanumeric characters and dashes\. Group names are case sensitive\.

1. You can keep the filter expression from the existing group, or optionally, enter a new filter expression\. For more information about how to build a filter expression, see [Using filter expressions to search for traces in the console](xray-console-filters.md)\. In the following example, the group filters for fault traces from the service `api.example.com`\. and requests to the service where the response time was greater than or equal to five seconds\.

   ```
   service("api.example.com") { fault = true OR responsetime >= 5 }
   ```

1. In **Tags**, edit tag keys and values, if needed\. Tag keys must be unique\. Tag values are optional; you can delete values if you want\. To delete a tag, choose **X** at the end of the tag's row\. For more information about tags, see [Tagging X\-Ray sampling rules and groups](xray-tagging.md)\.

1. Choose **Create group**\.

## Delete a group<a name="xray-console-group-delete"></a>

Follow steps in this section to delete a group in the X\-Ray console\. You can't delete the **Default** group\.

1. Sign in to the AWS Management Console and open the X\-Ray console at [https://console\.aws\.amazon\.com/xray/home](https://console.aws.amazon.com/xray/home)\.

1. To delete a group from the **Edit group** page, do the following\.

   1. On the **Groups** page, choose a group name to edit the group\.

   1. Choose **Delete**\.

   1. When you are prompted to confirm, choose **Delete**\.

1. To delete a group from the **Actions** menu, do the following\.

   1. On the **Groups** page, select a group by selecting its check box\. You can only select one group at a time\.

   1. On the **Actions** menu, choose **Delete group**\.  
![\[Actions menu, Delete group option\]](http://docs.aws.amazon.com/xray/latest/devguide/images/group-delete-actions-menu.png)

   1. When you're prompted to confirm, choose **Delete**\.

## View group metrics in Amazon CloudWatch<a name="xray-console-group-cloudwatch"></a>

After a group is created, incoming traces are checked against the group’s filter expression as they're stored in the X\-Ray service\. Metrics for the number of traces matching each criteria are published to Amazon CloudWatch every minute\. Choosing **View metric** on the **Edit group** page opens the CloudWatch console to the **Metric** page\. For more information about how to use CloudWatch metrics, see [Using Amazon CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) in the *Amazon CloudWatch User Guide*\.

1. Sign in to the AWS Management Console and open the X\-Ray console at [https://console\.aws\.amazon\.com/xray/home](https://console.aws.amazon.com/xray/home)\.

1. Do one of the following to open the **Edit group** page\.

   1. On the **Groups** page, choose a group to edit it\.

   1. On the group menu on one of the following pages, point to a group, and then choose **Edit**\.
      + **Service map**
      + **Traces**
      + **Analytics**

1. On the **Edit group** page, choose **View metric**\.  
![\[Edit group page, View metric button\]](http://docs.aws.amazon.com/xray/latest/devguide/images/group-view-metric.png)

   The CloudWatch console **Metrics** page opens in a new tab\.