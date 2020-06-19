# Debugging CloudWatch synthetics canaries using X\-Ray<a name="xray-services-cloudwatch-synthetics"></a>

CloudWatch Synthetics is a fully managed service that enables you to monitor your endpoints and APIs using scripted canaries that run 24 hours per day, once per minute\. 

You can customize canary scripts to check for changes in: 
+ Availability
+ Latency
+ Transactions
+ Broken or dead links
+ Step\-by\-step task completions
+ Page load errors
+ Load Latencies for UI assets
+ Complex wizard flows
+ Checkout flows in your application

Canaries follow the same routes and perform the same actions and behaviors as your customers, and continually verify the customer experience\.

To learn more about setting up Synthetics tests, see [Using Synthetics to Create and Manage Canaries](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html)\.

![\[Example canary node in x-ray service map.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-show-canary.png)

The following examples show common use cases for debugging issues that your Synthetics canaries raise\. Each example demonstrates a key strategy for debugging using either the service map or the X\-Ray Analytics console\.

For more information about how to read and interact with the service map, see [Viewing the Service Map](https://docs.aws.amazon.com/xray/latest/devguide/xray-console.html#xray-console-servicemap)\. 

For more information about how to read and interact with the X\-Ray Analytics console, see [Interacting with the AWS X\-Ray Analytics Console](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-analytics.html)\. 

**Topics**
+ [View canaries with increased error reporting in the service map](#xray-services-cloudwatch-synthetics-workflows-which-canary)
+ [Use trace maps for individual traces to view each request in detail](#xray-services-cloudwatch-synthetics-workflows-trace-map)
+ [Determine the root cause of ongoing failures in upstream and downstream services](#xray-services-cloudwatch-synthetics-workflows-root-cause)
+ [Identify performance bottlenecks and trends](#xray-services-cloudwatch-synthetics-workflows-bottlenecks)
+ [Compare latency and error or fault rates before and after changes](#xray-services-cloudwatch-synthetics-workflows-latency)
+ [Determine the required canary coverage for all APIs and URLs](#xray-services-cloudwatch-synthetics-workflows-impact)
+ [Use groups to focus on synthetics tests](#xray-services-cloudwatch-synthetics-groups)

## View canaries with increased error reporting in the service map<a name="xray-services-cloudwatch-synthetics-workflows-which-canary"></a>

To see which canaries have an increase in errors, faults, throttling rates, or slow response times within your X\-Ray service map, you can highlight Synthetics canary client nodes using the `Client::Synthetic` [filter](xray-console-filters.md)\. Clicking a Synthetics canary node displays the response time distribution of the entire request\. 

![\[Example canary node in x-ray service map with edge details.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-edgedetail.png)

## Use trace maps for individual traces to view each request in detail<a name="xray-services-cloudwatch-synthetics-workflows-trace-map"></a>

To determine which service results in the most latency or is causing an error, invoke the trace map by selecting the trace in the service map\. Individual trace maps display the end\-to\-end path of a single request\. Use this to understand the services invoked, and visualize the upstream and downstream services\. 

![\[Example canary node in x-ray trace map.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-tracemap.png)

## Determine the root cause of ongoing failures in upstream and downstream services<a name="xray-services-cloudwatch-synthetics-workflows-root-cause"></a>

Once you receive a CloudWatch alarm for failures in a Synthetics canary, use the statistical modeling on trace data in X\-Ray to determine the probable root cause of the issue within the X\-Ray Analytics console\. In the Analytics console, the **Response Time Root Cause** table shows recorded entity paths\. X\-Ray determines which path in your trace is the most likely cause for the response time\. The format indicates a hierarchy of entities that are encountered, ending in a response time root cause\. 

The following example shows that the Synthetics test for API “XXX” running on API Gateway is failing due to a throughput capacity exception from the Amazon DynamoDB table\.

![\[Example canary node in x-ray service map.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-select.png)

![\[Example canary node root cause.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-rootcause.png)

![\[Example annotation filter intdicating the canary node.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-showannot.png)

## Identify performance bottlenecks and trends<a name="xray-services-cloudwatch-synthetics-workflows-bottlenecks"></a>

You can view trends in the performance of your endpoint over time using continuous traffic from your Synthetics canaries to populate a trace map over a period of time\. 

![\[Example annotation filter intdicating the canary node.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-distribution.png)

## Compare latency and error or fault rates before and after changes<a name="xray-services-cloudwatch-synthetics-workflows-latency"></a>

Pinpoint the time a change occured to correlate that change to an increase in issues caught by your canaries\. Use the X\-Ray Analytics console to define the before and after time ranges as different trace sets, creating a visual differentiation in the response time distribution\.

![\[Example annotation filter intdicating the canary node.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-compare.png)

## Determine the required canary coverage for all APIs and URLs<a name="xray-services-cloudwatch-synthetics-workflows-impact"></a>

 Use X\-Ray Analytics to compare the experience of canaries with the users\. The UI below shows a blue trend line for canaries and a green line for the users\. You can also identify that two out of the three URLs don’t have canary tests\.

![\[Example annotation filter intdicating the canary node.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-vs-customer.png)

## Use groups to focus on synthetics tests<a name="xray-services-cloudwatch-synthetics-groups"></a>

To focus on a certain set of workflows, such as a Synthetics tests for application “www” running on AWS Elastic Beanstalk\. You can create an X\-Ray group using a filter expression\.

**Example Group filter expression**  

```
"edge(id(name: "www", type: "client::Synthetics"), id(name: "www", type: "AWS::ElasticBeanstalk::Environment"))" 
```

![\[Example nodes for Elastic Beanstalk www.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/synthetics-canary-www.png)