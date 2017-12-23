# Configuring the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-configuration"></a>

You can configure the X\-Ray SDK for \.NET with plugins to include information about the service that your application runs on, modify the default sampling behavior, or add sampling rules that apply to requests to specific paths\.


+ [Plugins](#xray-sdk-dotnet-configuration-plugins)
+ [Sampling Rules](#xray-sdk-dotnet-configuration-sampling)
+ [Logging](#xray-sdk-dotnet-configuration-logging)
+ [Environment Variables](#xray-sdk-dotnet-configuration-envvars)

## Plugins<a name="xray-sdk-dotnet-configuration-plugins"></a>

Use plugins to add data about the service hosting your application\.

**Plugins**

+ Amazon EC2 – `EC2Plugin` adds the instance ID and Availability Zone\.

To use a plugin, configure the X\-Ray SDK for \.NET client by adding the `AWSXRayPlugins` setting\.

**Example Web\.config \- plugins**  

```
<configuration>
  <appSettings>
    <add key="AWSXRayPlugins" value="EC2Plugin"/>
  </appSettings>
</configuration>
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

Tell the X\-Ray SDK for \.NET to load sampling rules from a file with the `SamplingRuleManifest` setting\.

**Example Web\.config \- Sampling Rules**  

```
<configuration>
  <appSettings>
    <add key="SamplingRuleManifest" value="sampling-rules.json"/>
  </appSettings>
</configuration>
```

## Logging<a name="xray-sdk-dotnet-configuration-logging"></a>

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

## Environment Variables<a name="xray-sdk-dotnet-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for \.NET\. The SDK supports the following variables\.

+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the service name that you set on the servlet filter's segment naming strategy\.

+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to listen on a different port or if it is running on a different host\.

+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**

  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.

  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.