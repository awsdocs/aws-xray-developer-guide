# Configuring the X\-Ray SDK for Python<a name="xray-sdk-python-configuration"></a>

The X\-Ray SDK for Python has a class named `xray_recorder` that provides the global recorder\. You can configure the global recorder to customize the middleware that creates segments for incoming HTTP calls\.

**Topics**
+ [Service Plugins](#xray-sdk-python-configuration-plugins)
+ [Sampling Rules](#xray-sdk-python-configuration-sampling)
+ [Logging](#xray-sdk-python-configuration-logging)
+ [Recorder Configuration in Code](#xray-sdk-python-middleware-configuration-code)
+ [Recorder Configuration with Django](#xray-sdk-python-middleware-configuration-django)
+ [Environment Variables](#xray-sdk-python-configuration-envvars)

## Service Plugins<a name="xray-sdk-python-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**
+ Amazon EC2 – `EC2Plugin` adds the instance ID and Availability Zone\.
+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ECSPlugin` adds the container ID\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-segment-resources-python09.png)

To use a plugin, call `configure` on the `xray_recorder`\.

```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

xray_recorder.configure(aws_xray_tracing_name='My app')
plugins = ('ElasticBeanstalkPlugin', 'EC2Plugin')
xray_recorder.configure(plugins=plugins)
patch_all()
```

You can also use [environment variables](#xray-sdk-python-configuration-envvars), which take precedence over values set in code, to configure the recorder\.

Configure plugins before [patching libraries](#xray-sdk-python-configuration) to record downstream calls\.

The SDK also uses plugin settings to set the `origin` field on the segment\. This indicates the type of AWS resource that runs your application\. The resource type appears under your application's name in the service map\. For example, `AWS::ElasticBeanstalk::Environment`\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the plugin that was loaded last to determine the origin\.

## Sampling Rules<a name="xray-sdk-python-configuration-sampling"></a>

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

To configure sampling rules, call `xray_recorder.configure`, as shown in the following example, where *rules* is either a dictionary of rules or the absolute path to a JSON file containing sampling rules\.

```
xray_recorder.configure(sampling_rules=rules)
```

You can also configure the global recorder to disable sampling and instrument all incoming requests\.

**Example main\.py – disable sampling**  

```
xray_recorder.configure(sampling=False)
```

## Logging<a name="xray-sdk-python-configuration-logging"></a>

The SDK uses Python’s built\-in `logging` module\. Get a reference to the logger for the `aws_xray_sdk` class and call `setLevel` on it to configure the different log level for the library and the rest of your application\.

**Example app\.py – logging**  

```
logging.basicConfig(level='WARNING')
logging.getLogger('aws_xray_sdk').setLevel(logging.DEBUG)
```

Use debug logs to identify issues, such as unclosed subsegments, when you [generate subsegments manually](xray-sdk-python-subsegments.md)\.

## Recorder Configuration in Code<a name="xray-sdk-python-middleware-configuration-code"></a>

Additional settings are available from the `configure` method on `xray_recorder`\.
+ `context_missing` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.
+ `daemon_address` – Set the host and port of the X\-Ray daemon listener\.
+ `service` – Set a service name that the SDK uses for segments\.
+ `plugins` – Record information about your application's AWS resources\.
+ `sampling` – Set to `False` to disable sampling\.
+ `sampling_rules` – Set the path of the JSON file containing your [sampling rules](#xray-sdk-python-configuration-sampling)\.

**Example main\.py – disable context missing exceptions**  

```
from aws_xray_sdk.core import xray_recorder

xray_recorder.configure(context_missing='LOG_ERROR')
```

## Recorder Configuration with Django<a name="xray-sdk-python-middleware-configuration-django"></a>

If you use the Django framework, you can use the Django `settings.py` file to configure options on the global recorder\.
+ `AUTO_INSTRUMENT` \(Django only\) – Record subsegments for built\-in database and template rendering operations\.
+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\.
+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\.
+ `PLUGINS` – Record information about your application's AWS resources\.
+ `SAMPLING` – Set to `False` to disable sampling\.
+ `SAMPLING_RULES` – Set the path of the JSON file containing your [sampling rules](#xray-sdk-python-configuration-sampling)\.

To enable recorder configuration in `settings.py`, add the Django middleware to the list of installed apps\.

**Example settings\.py – installed apps**  

```
INSTALLED_APPS = [
    ...
    'django.contrib.sessions',
    'aws_xray_sdk.ext.django',
]
```

Configure the available settings in a dict named `XRAY_RECORDER`\.

**Example settings\.py – installed apps**  

```
XRAY_RECORDER = {
    'AUTO_INSTRUMENT': True,
    'AWS_XRAY_CONTEXT_MISSING': 'LOG_ERROR',
    'AWS_XRAY_DAEMON_ADDRESS': '127.0.0.1:5000',
    'AWS_XRAY_TRACING_NAME': 'My application',
    'PLUGINS': ('ElasticBeanstalkPlugin', 'EC2Plugin', 'ECSPlugin'),
    'SAMPLING': False,
}
```

## Environment Variables<a name="xray-sdk-python-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Python\. The SDK supports the following variables: 
+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the service name that you set on the servlet filter's [segment naming strategy](xray-sdk-python-middleware.md#xray-sdk-python-middleware-naming)\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.
+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**
  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.
  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.

Environment variables override values set in code\.
