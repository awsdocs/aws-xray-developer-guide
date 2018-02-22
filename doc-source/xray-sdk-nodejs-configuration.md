# Configuring the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-configuration"></a>

You can configure the X\-Ray SDK for Node\.js with plugins to include information about the service that your application runs on, modify the default sampling behavior, or add sampling rules that apply to requests to specific paths\.


+ [Service Plugins](#xray-sdk-nodejs-configuration-plugins)
+ [Sampling Rules](#xray-sdk-nodejs-configuration-sampling)
+ [Logging](#xray-sdk-nodejs-configuration-logging)
+ [X\-Ray Daemon Address](#xray-sdk-nodejs-configuration-daemon)
+ [Environment Variables](#xray-sdk-nodejs-configuration-envvars)

## Service Plugins<a name="xray-sdk-nodejs-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**

+ Amazon EC2 – `EC2Plugin` adds the instance ID and Availability Zone\.

+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.

+ Amazon ECS – `ECSPlugin` adds the container ID\.

To use a plugin, configure the X\-Ray SDK for Node\.js client by using the `config` method\.

**Example app\.js \- Plugins**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.config([AWSXRay.plugins.EC2Plugin,AWSXRay.plugins.ElasticBeanstalkPlugin]);
```

The SDK also uses plugin settings to set the `origin` field on the segment\. This indicates the type of AWS resource that runs your application\. The resource type appears under your application's name in the service map\. For example, `AWS::ElasticBeanstalk::Environment`\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the plugin that was loaded last to determine the origin\.

## Sampling Rules<a name="xray-sdk-nodejs-configuration-sampling"></a>

The SDK has a default sampling strategy that determines which requests get traced\. By default, the SDK traces the first request each second, and five percent of any additional requests\. You can customize the SDK's sampling behavior by applying rules you define in a local file\.

**Example sampling\-rules\.json**  

```
{
  "version": 1,
  "rules": [
    {
      "description": "Player moves.",
      "service_name": "*",
      "http_method": "*",
      "url_path": "/api/move/*",
      "fixed_target": 0,
      "rate": 0.05
    }
  ],
  "default": {
    "fixed_target": 1,
    "rate": 0.1
  }
}
```

This example defines one custom rule and a default rule\. The custom rule applies a five\-percent sampling rate with no minimum number of requests to trace for paths under `/api/move/`\. The default rule traces the first request each second and 10 percent of additional requests\.

The SDK applies custom rules in the order in which they are defined\. If a request matches multiple custom rules, the SDK applies only the first rule\.

On Lambda, you cannot modify the sampling rate\. If your function is called by an instrumented service, calls generated requests that were sampled by that service will be recorded by Lambda\. If active tracing is enabled and no tracing header is present, Lambda makes the sampling decision\.

Tell the X\-Ray SDK for Node\.js to load sampling rules from a file with `setSamplingRules`\.

**Example app\.js \- Sampling rules from a file**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.middleware.setSamplingRules('sampling-rules.json');
```

You can also define your rules in code and pass them to `setSamplingRules` as an object\.

**Example app\.js \- Sampling rules from an object**  

```
var AWSXRay = require('aws-xray-sdk');
var rules = {
  "rules": [ { "description": "Player moves.", "service_name": "*", "http_method": "*", "url_path": "/api/move/*", "fixed_target": 0, "rate": 0.05 } ],
  "default": { "fixed_target": 1, "rate": 0.1 },
  "version": 1
  }

AWSXRay.middleware.setSamplingRules(rules);
```

## Logging<a name="xray-sdk-nodejs-configuration-logging"></a>

 To log output from the SDK, call `AWSXRay.setLogger(logger)`, where `logger` is an object that provides standard logging methods \(`warn`, `info`, etc\.\)\.

**Example app\.js \- Logging with Winston**  

```
var AWSXRay = require('aws-xray-sdk');
var logger = require('winston');
AWSXRay.setLogger(logger);
AWSXRay.config([AWSXRay.plugins.EC2Plugin]);
```

Call `setLogger` before you run other configuration methods to ensure that you capture output from those operations\.

To configure the SDK to output logs to the console without using a logging library, use the `AWS_XRAY_DEBUG_MODE` [environment variable](#xray-sdk-nodejs-configuration-envvars)\.

## X\-Ray Daemon Address<a name="xray-sdk-nodejs-configuration-daemon"></a>

If the X\-Ray daemon listens on a port or host other than `127.0.0.1:2000`, you can configure the X\-Ray SDK for Node\.js to send trace data to a different UDP address\.

```
AWSXRay.setDaemonAddress('host:port');
```

You can specify the host by name or by IPv4 address\.

**Example app\.js \- Daemon address**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.setDaemonAddress('daemonhost:8082');
```

You can also set the daemon address by using the `AWS_XRAY_DAEMON_ADDRESS` [environment variable](#xray-sdk-nodejs-configuration-envvars)\.

## Environment Variables<a name="xray-sdk-nodejs-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Node\.js\. The SDK supports the following variables\.

+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the segment name that you [set on the Express middleware](xray-sdk-nodejs-middleware.md)\.

+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.

+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**

  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.

  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.

+ `AWS_XRAY_DEBUG_MODE` – Set to `TRUE` to configure the SDK to output logs to the console, instead of [configuring a logger](#xray-sdk-nodejs-configuration-logging)\.