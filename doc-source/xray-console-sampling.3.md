# Configuring sampling rules in the X\-Ray console<a name="xray-console-sampling"></a>

You can use the AWS X\-Ray console to configure sampling rules for your services\. The X\-Ray SDK and AWS services that support [active tracing](xray-usage.md#xray-usage-services) with sampling configuration use sampling rules to determine which requests to record\. 

**Topics**
+ [Configuring sampling rules](#xray-console-config)
+ [Customizing sampling rules](#xray-console-custom)
+ [Sampling rule options](#xray-console-sampling-options)
+ [Sampling rule examples](#xray-console-sampling-examples)
+ [Configuring your service to use sampling rules](#xray-console-sampling-service)
+ [Viewing sampling results](#xray-console-sampling-results)
+ [Next steps](#xray-console-sampling-nextsteps)

## Configuring sampling rules<a name="xray-console-config"></a>

You can configure sampling for the following use cases:
+ **API Gateway Entrypoint** – API Gateway supports sampling and active tracing\. To enable active tracing on an API stage, see [Amazon API Gateway active tracing support for AWS X\-Ray](xray-services-apigateway.md)\.
+ **AWS AppSync** – AWS AppSync supports sampling and active tracing\. To enable active tracing on AWS AppSync requests, see [Tracing with AWS X\-Ray](https://docs.aws.amazon.com/appsync/latest/devguide/x-ray-tracing.html)\.
+ **Instrument X\-Ray SDK on compute platforms** – When using compute platforms such as Amazon EC2, Amazon ECS, or AWS Elastic Beanstalk, sampling is supported when the application has been instrumented with the lastest X\-Ray SDK\.

## Customizing sampling rules<a name="xray-console-custom"></a>

By customizing sampling rules, you can control the amount of data that you record, and modify sampling behavior on the fly without modifying or redeploying your code\. Sampling rules tell the X\-Ray SDK how many requests to record for a set of criteria\. By default, the X\-Ray SDK records the first request each second, and five percent of any additional requests\. One request per second is the *reservoir*\. This ensures that at least one trace is recorded each second as long as the service is serving requests\. Five percent is the *rate* at which additional requests beyond the reservoir size are sampled\.

You can configure the X\-Ray SDK to read sampling rules from a JSON document that you include with your code\. However, when you run multiple instances of your service, each instance performs sampling independently\. This causes the overall percentage of requests sampled to increase because the reservoirs of all of the instances are effectively added together\. Additionally, to update local sampling rules, you need to redeploy your code\.

By defining sampling rules in the X\-Ray console, and [configuring the SDK](#xray-console-sampling-service) to read rules from the X\-Ray service, you can avoid both of these issues\. The service manages the reservoir for each rule, and assigns quotas to each instance of your service to distribute the reservoir evenly, based on the number of instances that are running\. The reservoir limit is calculated according to the rules you set\. And because the rules are configured in the service, you can manage rules without making additional deployments\.

**To configure sampling rules in the X\-Ray console**

1. Open the [X\-Ray console](https://console.aws.amazon.com/xray/home#/service-map)\.

1. Choose **Sampling**\.

1. To create a rule, choose **Create sampling rule**\.

   To edit a rule, choose a rule's name\.

   To delete a rule, choose a rule and use the **Actions** menu to delete it\.

## Sampling rule options<a name="xray-console-sampling-options"></a>

The following options are available for each rule\. String values can use wildcards to match a single character \(`?`\) or zero or more characters \(`*`\)\.

**Sampling rule options**
+ **Rule name** \(string\) – A unique name for the rule\.
+ **Priority** \(integer between 1 and 9999\) – The priority of the sampling rule\. Services evaluate rules in ascending order of priority, and make a sampling decision with the first rule that matches\.
+ **Reservoir** \(non\-negative integer\) – A fixed number of matching requests to instrument per second, before applying the fixed rate\. The reservoir is not used directly by services, but applies to all services using the rule collectively\.
+ **Rate** \(number between 0 and 100\) – The percentage of matching requests to instrument, after the reservoir is exhausted\. The rate may be an integer or a float\.
+ **Service name** \(string\) – The name of the instrumented service, as it appears in the service map\.
  + X\-Ray SDK – The service name that you configure on the recorder\.
  + Amazon API Gateway – `api-name/stage`\.
+ **Service type** \(string\) – The service type, as it appears in the service map\. For the X\-Ray SDK, set the service type by applying the appropriate plugin:
  + `AWS::ElasticBeanstalk::Environment` – An AWS Elastic Beanstalk environment \(plugin\)\.
  + `AWS::EC2::Instance` – An Amazon EC2 instance \(plugin\)\.
  + `AWS::ECS::Container` – An Amazon ECS container \(plugin\)\.
  + `AWS::APIGateway::Stage` – An Amazon API Gateway stage\.
  + `AWS::AppSync::GraphQLAPI ` – An AWS AppSync API request\.
+ **Host** \(string\) – The hostname from the HTTP host header\.
+ **HTTP method** \(string\) – The method of the HTTP request\.
+ **URL path** \(string\) – The URL path of the request\.
  + X\-Ray SDK – The path portion of the HTTP request URL\.
  + Amazon API Gateway – Not supported\.
+ **Resource ARN** \(string\) – The ARN of the AWS resource running the service\.
  + X\-Ray SDK – Not supported\. The SDK can only use rules with **Resource ARN** set to `*`\.
  + Amazon API Gateway – The stage ARN\.
+ \(Optional\) **Attributes** \(key and value\) – Segment attributes that are known when the sampling decision is made\.
  + X\-Ray SDK – Not supported\. The SDK ignores rules that specify attributes\.
  + Amazon API Gateway – Headers from the original HTTP request\.

## Sampling rule examples<a name="xray-console-sampling-examples"></a>

**Example – Default rule with no reservoir and a low rate**  
You can modify the default rule's reservoir and rate\. The default rule applies to requests that don't match any other rule\.  
+ **Reservoir** – **0**
+ **Rate** – **0\.005** \(0\.5 percent\)

**Example – Debugging rule to trace all requests for a problematic route**  
A high\-priority rule applied temporarily for debugging\.  
+ **Rule name** – **DEBUG – history updates**
+ **Priority** – **1**
+ **Reservoir** – **1**
+ **Rate** – **1**
+ **Service name** – **Scorekeep**
+ **Service type** – **\***
+ **Host** – **\***
+ **HTTP method** – **PUT**
+ **URL path** – **/history/\***
+ **Resource ARN** – **\***

**Example – Higher minimum rate for POSTs**  
+ **Rule name** – **POST minimum**
+ **Priority** – **100**
+ **Reservoir** – **10**
+ **Rate** – **0\.10**
+ **Service name** – **\***
+ **Service type** – **\***
+ **Host** – **\***
+ **HTTP method** – **POST**
+ **URL path** – **\***
+ **Resource ARN** – **\***

## Configuring your service to use sampling rules<a name="xray-console-sampling-service"></a>

The X\-Ray SDK requires additional configuration to use sampling rules that you configure in the console\. See the configuration topic for your language for details on configuring a sampling strategy:
+ Java – [Sampling rules](xray-sdk-java-configuration.md#xray-sdk-java-configuration-sampling)
+ Go – [Sampling rules](xray-sdk-go-configuration.md#xray-sdk-go-configuration-sampling)
+ Node\.js – [Sampling rules](xray-sdk-nodejs-configuration.md#xray-sdk-nodejs-configuration-sampling)
+ Python – [Sampling rules](xray-sdk-python-configuration.md#xray-sdk-python-configuration-sampling)
+ Ruby – [Sampling rules](xray-sdk-ruby-configuration.md#xray-sdk-ruby-configuration-sampling)
+ \.NET – [Sampling rules](xray-sdk-dotnet-configuration.md#xray-sdk-dotnet-configuration-sampling)

For API Gateway, see [Amazon API Gateway active tracing support for AWS X\-Ray](xray-services-apigateway.md)\.

## Viewing sampling results<a name="xray-console-sampling-results"></a>

The X\-Ray console **Sampling** page shows detailed information about how your services use each sampling rule\.

The **Trend** column shows how the rule has been used in the last few minutes\. Each column shows statistics for a 10\-second window\.

**Sampling statistics**
+ **Total matched rule** – The number of requests that matched this rule\. This number doesn't include requests that could have matched this rule, but matched a higher\-priority rule first\.
+ **Total sampled** – The number of requests recorded\.
+ **Sampled with fixed rate** – The number of requests sampled by applying the rule's fixed rate\.
+ **Sampled with reservoir limit** – The number of requests sampled using a quota assigned by X\-Ray\.
+ **Borrowed from reservoir** – The number of requests sampled by borrowing from the reservoir\. The first time a service matches a request to a rule, it has not yet been assigned a quota by X\-Ray\. However, if the reservoir is at least 1, the service borrows one trace per second until X\-Ray assigns a quota\.

For more information about sampling statistics and how services use sampling rules, see [Using sampling rules with the X\-Ray API](xray-api-sampling.md)\.

## Next steps<a name="xray-console-sampling-nextsteps"></a>

You can use the X\-Ray API to manage sampling rules\. With the API, you can create and update rules programmatically on a schedule, or in response to alarms or notifications\. See [Configuring  sampling, groups, and encryption settings with the AWS X\-Ray API](xray-api-configuration.md) for instructions and additional rule examples\.

The X\-Ray SDK and AWS services also use the X\-Ray API to read sampling rules, report sampling results, and get sampling targets\. Services must keep track of how often they apply each rule, evaluate rules based on priority, and borrow from the reservoir when a request matches a rule for which X\-Ray has not yet assigned the service a quota\. For more detail about how a service uses the API for sampling, see [Using sampling rules with the X\-Ray API](xray-api-sampling.md)\.

When the X\-Ray SDK calls sampling APIs, it uses the X\-Ray daemon as a proxy\. If you already use TCP port 2000, you can configure the daemon to run the proxy on a different port\. See [Configuring the AWS X\-Ray daemon](xray-daemon-configuration.md) for details\.