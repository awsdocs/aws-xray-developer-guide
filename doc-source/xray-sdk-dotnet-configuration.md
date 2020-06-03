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

**Example \.NET appsettings\.json**  

```
{
  "XRay": {
    "AWSXRayPlugins": "EC2Plugin",
    "SamplingRuleManifest": "sampling-rules.json"
  }
}
```

Then, in your application code, build a configuration object and use it to initialize the X\-Ray recorder\. Do this before you [initialize the recorder](xray-sdk-dotnet-messagehandler.md#xray-sdk-dotnet-messagehandler-startupcs)\.

**Example \.NET Core Program\.cs – Recorder configuration**  

```
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
...
AWSXRayRecorder.InitializeInstance(configuration);
```

If you are instrumenting a \.NET Core web application, you can also pass the configuration object to the `UseXRay` method when you [configure the message handler](xray-sdk-dotnet-messagehandler.md#xray-sdk-dotnet-messagehandler-startupcs)\. For Lambda functions, use the `InitializeInstance` method as shown above\.

For more information on the \.NET Core configuration API, see [Configure an ASP\.NET Core App](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?tabs=basicconfiguration) on docs\.microsoft\.com\.

**Topics**
+ [Plugins](#xray-sdk-dotnet-configuration-plugins)
+ [Sampling rules](#xray-sdk-dotnet-configuration-sampling)
+ [Logging \(\.NET\)](#xray-sdk-dotnet-configuration-logging)
+ [Logging \(\.NET Core\)](#xray-sdk-dotnet-configuration-corelogging)
+ [Environment variables](#xray-sdk-dotnet-configuration-envvars)

## Plugins<a name="xray-sdk-dotnet-configuration-plugins"></a>

Use plugins to add data about the service that is hosting your application\.

**Plugins**
+ Amazon EC2 – `EC2Plugin` adds the instance ID, Availability Zone, and the CloudWatch Logs Group\.
+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ECSPlugin` adds the container ID\.

To use a plugin, configure the X\-Ray SDK for \.NET client by adding the `AWSXRayPlugins` setting\. If multiple plugins apply to your application, specify all of them in the same setting, separated by commas\.

**Example Web\.config \- plugins**  

```
<configuration>
  <appSettings>
    <add key="AWSXRayPlugins" value="EC2Plugin,ElasticBeanstalkPlugin"/>
  </appSettings>
</configuration>
```

**Example \.NET Core appsettings\.json – Plugins**  

```
{
  "XRay": {
    "AWSXRayPlugins": "EC2Plugin,ElasticBeanstalkPlugin"
  }
}
```

## Sampling rules<a name="xray-sdk-dotnet-configuration-sampling"></a>

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

To configure backup rules, tell the X\-Ray SDK for \.NET to load sampling rules from a file with the `SamplingRuleManifest` setting\.

**Example \.NET Web\.config \- sampling rules**  

```
<configuration>
  <appSettings>
    <add key="SamplingRuleManifest" value="sampling-rules.json"/>
  </appSettings>
</configuration>
```

**Example \.NET Core appsettings\.json – Sampling rules**  

```
{
  "XRay": {
    "SamplingRuleManifest": "sampling-rules.json"
  }
}
```

To use only local rules, build the recorder with a `LocalizedSamplingStrategy`\. If you have backup rules configured, remove that configuration\.

**Example \.NET global\.asax – Local sampling rules**  

```
var recorder = new AWSXRayRecorderBuilder().WithSamplingStrategy(new LocalizedSamplingStrategy(samplingrules.json)).Build();
AWSXRayRecorder.InitializeInstance(recorder);
```

**Example \.NET Core Program\.cs – Local sampling rules**  

```
var recorder = new AWSXRayRecorderBuilder().WithSamplingStrategy(new LocalizedSamplingStrategy(sampling-rules.json)).Build();
AWSXRayRecorder.InitializeInstance(configuration,recorder);
```

## Logging \(\.NET\)<a name="xray-sdk-dotnet-configuration-logging"></a>

The X\-Ray SDK for \.NET uses the same logging mechanism as the AWS SDK for \.NET\. If you already configured your application to log AWS SDK for \.NET output, the same configuration applies to output from the X\-Ray SDK for \.NET\.

To configure logging, add a configuration section named `aws` to your `App.config` file or `Web.config` file\.

**Example Web\.config \- logging**  

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

For more information, see [Configuring Your AWS SDK for \.NET Application](https://docs.aws.amazon.com/sdk-for-net/latest/developer-guide/net-dg-config.html) in the *AWS SDK for \.NET Developer Guide*\.

## Logging \(\.NET Core\)<a name="xray-sdk-dotnet-configuration-corelogging"></a>

For \.NET Core applications, the X\-Ray SDK supports the logging options in the AWS SDK for \.NET [LoggingOptions enum](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/Amazon/TLoggingOptions.html)\. To configure logging, pass one of these options to the `RegisterLogger` method\.

```
AWSXRayRecorder.RegisterLogger(LoggingOptions.Console);
```

For example, to use log4net, create a configuration file that defines the logger, the output format, and the file location\.

**Example \.NET Core log4net\.config**  

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

**Example \.NET Core Program\.cs – Logging**  

```
using log4net;
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);

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

## Environment variables<a name="xray-sdk-dotnet-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for \.NET\. The SDK supports the following variables\.
+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the service name that you set on the servlet filter's [segment naming strategy](xray-sdk-dotnet-messagehandler.md#xray-sdk-dotnet-messagehandler-naming)\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK uses `127.0.0.1:2000` for both trace data \(UDP\) and sampling \(TCP\)\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.

**Format**
  + **Same port** – `address:port`
  + **Different ports** – `tcp:address:port udp:address:port`
+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**
  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.
  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.