# Configuring the X\-Ray SDK for Ruby<a name="xray-sdk-ruby-configuration"></a>

The X\-Ray SDK for Ruby has a class named `XRay.recorder` that provides the global recorder\. You can configure the global recorder to customize the middleware that creates segments for incoming HTTP calls\.

**Topics**
+ [Service plugins](#xray-sdk-ruby-configuration-plugins)
+ [Sampling rules](#xray-sdk-ruby-configuration-sampling)
+ [Logging](#xray-sdk-ruby-configuration-logging)
+ [Recorder configuration in code](#xray-sdk-ruby-configuration-code)
+ [Recorder configuration with rails](#xray-sdk-ruby-middleware-configuration-rails)
+ [Environment variables](#xray-sdk-ruby-configuration-envvars)

## Service plugins<a name="xray-sdk-ruby-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**
+ Amazon EC2 – `ec2` adds the instance ID and Availability Zone\.
+ Elastic Beanstalk – `elastic_beanstalk` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ecs` adds the container ID\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-segment-resources-python09.png)

To use plugins, specify it in the configuration object that you pass to the recorder\.

**Example main\.rb – Plugin configuration**  

```
my_plugins = %I[ec2 elastic_beanstalk]

config = {
  plugins: my_plugins,
  name: 'my app',
}

XRay.recorder.configure(config)
```

You can also use [environment variables](#xray-sdk-ruby-configuration-envvars), which take precedence over values set in code, to configure the recorder\.

The SDK also uses plugin settings to set the `origin` field on the segment\. This indicates the type of AWS resource that runs your application\. The resource type appears under your application's name in the service map\. For example, `AWS::ElasticBeanstalk::Environment`\.

![\[Service node with resource type.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the following resolution order to determine the origin: ElasticBeanstalk > EKS > ECS > EC2\.

## Sampling rules<a name="xray-sdk-ruby-configuration-sampling"></a>

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

To configure backup rules, define a hash for the document in the configuration object that you pass to the recorder\.

**Example main\.rb – Backup rule configuration**  

```
require 'aws-xray-sdk'
my_sampling_rules =  {
  version: 1,
  default: {
    fixed_target: 1,
    rate: 0.1
  }
}
config = {
  sampling_rules: my_sampling_rules,
  name: 'my app',
}
XRay.recorder.configure(config)
```

To store the sampling rules independently, define the hash in a separate file and require the file to pull it into your application\.

**Example config/sampling\-rules\.rb**  

```
my_sampling_rules =  {
  version: 1,
  default: {
    fixed_target: 1,
    rate: 0.1
  }
}
```

**Example main\.rb – Sampling rule from a file**  

```
require 'aws-xray-sdk'
require config/sampling-rules.rb

config = {
  sampling_rules: my_sampling_rules,
  name: 'my app',
}
XRay.recorder.configure(config)
```

To use only local rules, require the sampling rules and configure the `LocalSampler`\. 

**Example main\.rb – Local rule sampling**  

```
require 'aws-xray-sdk'
require 'aws-xray-sdk/sampling/local/sampler'

config = {
  sampler: LocalSampler.new,
  name: 'my app',
}
XRay.recorder.configure(config)
```

You can also configure the global recorder to disable sampling and instrument all incoming requests\.

**Example main\.rb – Disable sampling**  

```
require 'aws-xray-sdk'
config = {
  sampling: false,
  name: 'my app',
}
XRay.recorder.configure(config)
```

## Logging<a name="xray-sdk-ruby-configuration-logging"></a>

By default, the recorder outputs info\-level events to `$stdout`\. You can customize logging by defining a [logger](https://ruby-doc.org/stdlib-2.4.2/libdoc/logger/rdoc/Logger.html) in the configuration object that you pass to the recorder\.

**Example main\.rb – Logging**  

```
require 'aws-xray-sdk'
config = {
  logger: my_logger,
  name: 'my app',
}
XRay.recorder.configure(config)
```

Use debug logs to identify issues, such as unclosed subsegments, when you [generate subsegments manually](xray-sdk-ruby-subsegments.md)\.

## Recorder configuration in code<a name="xray-sdk-ruby-configuration-code"></a>

Additional settings are available from the `configure` method on `XRay.recorder`\.
+ `context_missing` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.
+ `daemon_address` – Set the host and port of the X\-Ray daemon listener\.
+ `name` – Set a service name that the SDK uses for segments\.
+ `naming_pattern` – Set a domain name pattern to use [dynamic naming](xray-sdk-ruby-middleware.md#xray-sdk-ruby-middleware-naming)\.
+ `plugins` – Record information about your application's AWS resources with [plugins](#xray-sdk-ruby-configuration-plugins)\.
+ `sampling` – Set to `false` to disable sampling\.
+ `sampling_rules` – Set the hash containing your [sampling rules](#xray-sdk-ruby-configuration-sampling)\.

**Example main\.rb – Disable context missing exceptions**  

```
require 'aws-xray-sdk'
config = {
  context_missing: 'LOG_ERROR'
}

XRay.recorder.configure(config)
```

## Recorder configuration with rails<a name="xray-sdk-ruby-middleware-configuration-rails"></a>

If you use the Rails framework, you can configure options on the global recorder in a Ruby file under `app_root/initializers`\. The X\-Ray SDK supports an additional configuration key for use with Rails\.
+ `active_record` – Set to `true` to record subsegments for Active Record database transactions\.

Configure the available settings in a configuration object named `Rails.application.config.xray`\.

**Example config/initializers/aws\_xray\.rb**  

```
Rails.application.config.xray = {
  name: 'my app',
  patch: %I[net_http aws_sdk],
  active_record: true
}
```

## Environment variables<a name="xray-sdk-ruby-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Ruby\. The SDK supports the following variables: 
+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the service name that you set on the servlet filter's [segment naming strategy](xray-sdk-ruby-middleware.md#xray-sdk-ruby-middleware-naming)\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.
+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**
  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.
  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.

Environment variables override values set in code\.