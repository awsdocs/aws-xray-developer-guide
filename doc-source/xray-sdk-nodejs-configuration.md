# Configuring the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-configuration"></a>

You can configure the X\-Ray SDK for Node\.js with plugins to include information about the service that your application runs on, modify the default sampling behavior, or add sampling rules that apply to requests to specific paths\.

**Topics**
+ [Service plugins](#xray-sdk-nodejs-configuration-plugins)
+ [Sampling rules](#xray-sdk-nodejs-configuration-sampling)
+ [Logging](#xray-sdk-nodejs-configuration-logging)
+ [X\-Ray daemon address](#xray-sdk-nodejs-configuration-daemon)
+ [Environment variables](#xray-sdk-nodejs-configuration-envvars)

## Service plugins<a name="xray-sdk-nodejs-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**
+ Amazon EC2 – `EC2Plugin` adds the instance ID, Availability Zone, and the CloudWatch Logs Group\.
+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ECSPlugin` adds the container ID\.

To use a plugin, configure the X\-Ray SDK for Node\.js client by using the `config` method\.

**Example app\.js \- plugins**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.config([AWSXRay.plugins.EC2Plugin,AWSXRay.plugins.ElasticBeanstalkPlugin]);
```

The SDK also uses plugin settings to set the `origin` field on the segment\. This indicates the type of AWS resource that runs your application\. The resource type appears under your application's name in the service map\. For example, `AWS::ElasticBeanstalk::Environment`\.

![\[Service node with resource type.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the following resolution order to determine the origin: ElasticBeanstalk > EKS > ECS > EC2\.

## Sampling rules<a name="xray-sdk-nodejs-configuration-sampling"></a>

The SDK uses the sampling rules you define in the X\-Ray console to determine which requests to record\. The default rule traces the first request each second, and five percent of any additional requests across all services sending traces to X\-Ray\. [Create additional rules in the X\-Ray console](xray-console-sampling.md) to customize the amount of data recorded for each of your applications\.

The SDK applies custom rules in the order in which they are defined\. If a request matches multiple custom rules, the SDK applies only the first rule\.

**Note**  
If the SDK can't reach X\-Ray to get sampling rules, it reverts to a default local rule of the first request each second, and five percent of any additional requests per host\. This can occur if the host doesn't have permission to call sampling APIs, or can't connect to the X\-Ray daemon, which acts as a TCP proxy for API calls made by the SDK\.

You can also configure the SDK to load sampling rules from a JSON document\. The SDK can use local rules as a backup for cases where X\-Ray sampling is unavailable, or use local rules exclusively\.

**Example sampling\-rules\.json**  

```
{
  "version": 2,
  "rules": [
    {
      "description": "Player moves.",
      "host": "*",
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

The disadvantage of defining rules locally is that the fixed target is applied by each instance of the recorder independently, instead of being managed by the X\-Ray service\. As you deploy more hosts, the fixed rate is multiplied, making it harder to control the amount of data recorded\.

On AWS Lambda, you cannot modify the sampling rate\. If your function is called by an instrumented service, calls that generated requests that were sampled by that service will be recorded by Lambda\. If active tracing is enabled and no tracing header is present, Lambda makes the sampling decision\.

To configure backup rules, tell the X\-Ray SDK for Node\.js to load sampling rules from a file with `setSamplingRules`\.

**Example app\.js \- sampling rules from a file**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.middleware.setSamplingRules('sampling-rules.json');
```

You can also define your rules in code and pass them to `setSamplingRules` as an object\.

**Example app\.js \- sampling rules from an object**  

```
var AWSXRay = require('aws-xray-sdk');
var rules = {
  "rules": [ { "description": "Player moves.", "service_name": "*", "http_method": "*", "url_path": "/api/move/*", "fixed_target": 0, "rate": 0.05 } ],
  "default": { "fixed_target": 1, "rate": 0.1 },
  "version": 1
  }

AWSXRay.middleware.setSamplingRules(rules);
```

To use only local rules, call `disableCentralizedSampling`\.

```
AWSXRay.middleware.disableCentralizedSampling()
```

## Logging<a name="xray-sdk-nodejs-configuration-logging"></a>

 To log output from the SDK, call `AWSXRay.setLogger(logger)`, where `logger` is an object that provides standard logging methods \(`warn`, `info`, etc\.\)\.

**Example app\.js \- logging**  

```
var AWSXRay = require('aws-xray-sdk');

// Create your own logger, or instantiate one using a library.
var logger = {
  error: (message, meta) => { /* logging code */ },
  warn: (message, meta) => { /* logging code */ },
  info: (message, meta) => { /* logging code */ },
  debug: (message, meta) => { /* logging code */ }
}

AWSXRay.setLogger(logger);
AWSXRay.config([AWSXRay.plugins.EC2Plugin]);
```

Call `setLogger` before you run other configuration methods to ensure that you capture output from those operations\.

For a list of valid log level values, view the details in the [Environment variables](#xray-sdk-nodejs-configuration-envvars) section\.

To configure the SDK to output logs to the console without using a logging library, use the `AWS_XRAY_DEBUG_MODE` environment variable\.

## X\-Ray daemon address<a name="xray-sdk-nodejs-configuration-daemon"></a>

If the X\-Ray daemon listens on a port or host other than `127.0.0.1:2000`, you can configure the X\-Ray SDK for Node\.js to send trace data to a different address\.

```
AWSXRay.setDaemonAddress('host:port');
```

You can specify the host by name or by IPv4 address\.

**Example app\.js \- daemon address**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.setDaemonAddress('daemonhost:8082');
```

If you configured the daemon to listen on different ports for TCP and UDP, you can specify both in the daemon address setting\.

**Example app\.js \- daemon address on separate ports**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.setDaemonAddress('tcp:daemonhost:8082 udp:daemonhost:8083');
```

You can also set the daemon address by using the `AWS_XRAY_DAEMON_ADDRESS` [environment variable](#xray-sdk-nodejs-configuration-envvars)\.

## Environment variables<a name="xray-sdk-nodejs-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Node\.js\. The SDK supports the following variables\.
+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**
  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.
  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK uses `127.0.0.1:2000` for both trace data \(UDP\) and sampling \(TCP\)\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.

**Format**
  + **Same port** – `address:port`
  + **Different ports** – `tcp:address:port udp:address:port`
+ `AWS_XRAY_DEBUG_MODE` – Set to `TRUE` to configure the SDK to output logs to the console, instead of [configuring a logger](#xray-sdk-nodejs-configuration-logging)\.
+ `AWS_XRAY_LOG_LEVEL ` – Set a log level for the logger\. Valid values are `debug`, `info`, `warn`, `error`, and `silent`\. This value is ignored when AWS\_XRAY\_DEBUG\_MODE is set to `TRUE`\.
+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the segment name that you [set on the Express middleware](xray-sdk-nodejs-middleware.md)\.