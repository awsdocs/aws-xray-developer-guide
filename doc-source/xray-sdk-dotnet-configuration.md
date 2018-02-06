# Configuring the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-configuration"></a>

You can configure the X\-Ray SDK for \.NET with plugins to include information about the service that your application runs on, modify the default sampling behavior, or add sampling rules that apply to requests to specific paths\.

For \.NET web applications, add keys to the `appSettings` section of your `Web.config` file\.

**Example Web\.config**  

```
<configuration>
  <appSettings>
    <add key="AWSXRayPlugins" value="EC2Plugin"/>
    <add key="SamplingRuleManifest" value="sampling-rules.json"/>
  </appSettings>
</configuration>
```

For \.NET Core, create a file named `appsettings.json` with a top\-level key named `XRay`\.

**Example appsettings\.json**  

```
{
  "XRay": {
    "AWSXRayPlugins": "EC2Plugin",
    "SamplingRuleManifest": "sampling-rules.json"
  }
}
```

Then, in your application code, build a configuration object and use it to initialize the X\-Ray recorder\. Do this before you initialize the recorder\.

**Example Program\.cs – \.NET Core Configuration**  

```
using [Amazon\.XRay\.Recorder\.Core](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
...
AWSXRayRecorder.InitializeInstance(configuration);
```

If you are instrumenting a \.NET Core web application, you can also pass the configuration object to the `UseXRay` method when you configure the message handler\. For Lambda functions, use the `InitializeInstance` method as shown above\.

For more information on the \.NET Core configuration API, see [Configure an ASP\.NET Core App](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?tabs=basicconfiguration) on docs\.microsoft\.com\.


+ [Plugins](#xray-sdk-dotnet-configuration-plugins)
+ [Sampling Rules](#xray-sdk-dotnet-configuration-sampling)
+ [Logging \(\.NET\)](#xray-sdk-dotnet-configuration-logging)
+ [Logging \(\.NET Core\)](#xray-sdk-dotnet-configuration-corelogging)
+ [Environment Variables](#xray-sdk-dotnet-configuration-envvars)

## Plugins<a name="xray-sdk-dotnet-configuration-plugins"></a>

Use plugins to add data about the service that is hosting your application\.

**Plugins**

+ Amazon EC2 – `EC2Plugin` adds the instance ID and Availability Zone\.

+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.

+ Amazon ECS – `ECSPlugin` adds the container ID\.

To use a plugin, configure the X\-Ray SDK for \.NET client by adding the `AWSXRayPlugins` setting\. If multiple plugins apply to your application, specify all of them in the same setting, separated by commas\.

**Example Web\.config \- Plugins**  

```
<configuration>
  <appSettings>
    <add key="AWSXRayPlugins" value="EC2Plugin,ElasticBeanstalkPlugin"/>
  </appSettings>
</configuration>
```

**Example appsettings\.json – Plugins**  

```
{
  "XRay": {
    "AWSXRayPlugins": "EC2Plugin,ElasticBeanstalkPlugin"
  }
}
```

## Sampling Rules<a name="xray-sdk-dotnet-configuration-sampling"></a>

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

Tell the X\-Ray SDK for \.NET to load sampling rules from a file with the `SamplingRuleManifest` setting\.

**Example Web\.config \- Sampling Rules**  

```
<configuration>
  <appSettings>
    <add key="SamplingRuleManifest" value="sampling-rules.json"/>
  </appSettings>
</configuration>
```

**Example appsettings\.json – Sampling Rules**  

```
{
  "XRay": {
    "SamplingRuleManifest": "sampling-rules.json"
  }
}
```

## Logging \(\.NET\)<a name="xray-sdk-dotnet-configuration-logging"></a>

The X\-Ray SDK for \.NET uses the same logging mechanism as the AWS SDK for \.NET\. If you already configured your application to log AWS SDK for \.NET output, the same configuration applies to output from the X\-Ray SDK for \.NET\.

To configure logging, add a configuration section named `aws` to your `App.config` file or `Web.config` file\.

**Example Web\.config \- Logging**  

```
...
<configuration>
  <configSections>
    <section name="aws" type="Amazon.AWSSection, AWSSDK.Core"/>
  </configSections>
  <aws>
    <logging logTo="Log4Net"/>
  </aws>
</configuration>
```

For more information, see [Configuring Your AWS SDK for \.NET Application](http://docs.aws.amazon.com/sdk-for-net/latest/developer-guide/net-dg-config.html) in the *AWS SDK for \.NET Developer Guide*\.

## Logging \(\.NET Core\)<a name="xray-sdk-dotnet-configuration-corelogging"></a>

For \.NET Core applications, the X\-Ray SDK supports the logging options in the AWS SDK for \.NET [LoggingOptions enum](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/Amazon/TLoggingOptions.html)\. To configure logging, pass one of these options to the `RegisterLogger` method\.

```
AWSXRayRecorder.RegisterLogger(LoggingOptions.Console);
```

For example, to use log4net, create a configuration file that defines the logger, the output format, and the file location\.

**Example log4net\.config**  

```
<?xml version="1.0" encoding="utf-8" ?>
<log4net>
  <appender name="FileAppender" type="log4net.Appender.FileAppender,log4net">
    <file value="c:\logs\sdk-log.txt" />
    <layout type="log4net.Layout.PatternLayout">
      <conversionPattern value="%date [%thread] %level %logger - %message%newline" />
    </layout>
  </appender>
  <logger name="Amazon">
    <level value="DEBUG" />
    <appender-ref ref="FileAppender" />
  </logger>
</log4net>
```

Then, create the logger and apply the configuration in your program code\.

**Example Program\.cs – Logging**  

```
using log4net;
using [Amazon\.XRay\.Recorder\.Core](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);

class Program
{
  private static ILog log;
  static Program()
  {
    var logRepository = LogManager.GetRepository(Assembly.GetEntryAssembly());
    XmlConfigurator.Configure(logRepository, new FileInfo("log4net.config"));
    log = LogManager.GetLogger(typeof(Program));
    AWSXRayRecorder.RegisterLogger(LoggingOptions.Log4Net);
  }
  static void Main(string[] args)
  {
  ...
  }
}
```

For more information on configuring log4net, see [Configuration](https://logging.apache.org/log4net/release/manual/configuration.html) on logging\.apache\.org\.

## Environment Variables<a name="xray-sdk-dotnet-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for \.NET\. The SDK supports the following variables\.

+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the service name that you set on the servlet filter's segment naming strategy\.

+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to listen on a different port or if it is running on a different host\.

+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**

  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.

  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.