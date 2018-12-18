# Configuring the X\-Ray SDK for Go<a name="xray-sdk-go-configuration"></a>

You can specify the configuration for for X\-Ray SDK for Go through environment variables, by calling `Configure` with a `Config` object, or by assuming default values\. Environment variables take precedence over `Config` values, which take precedence over any default value\.

**Topics**
+ [Service Plugins](#xray-sdk-go-configuration-plugins)
+ [Sampling Rules](#xray-sdk-go-configuration-sampling)
+ [Logging](#xray-sdk-go-configuration-logging)
+ [Environment Variables](#xray-sdk-go-configuration-envvars)
+ [Using Configure](#xray-sdk-go-configuration-configure)

## Service Plugins<a name="xray-sdk-go-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**
+ Amazon EC2 – `EC2Plugin` adds the instance ID and Availability Zone\.
+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ECSPlugin` adds the container ID\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-segment-resources-go.png)

To use a plugin, import one of the following packages\.

```
_ "github.com/aws/aws-xray-sdk-go/plugins/ec2"
_ "github.com/aws/aws-xray-sdk-go/plugins/ecs"
_ "github.com/aws/aws-xray-sdk-go/plugins/beanstalk"
```

The SDK also uses plugin settings to set the `origin` field on the segment\. This indicates the type of AWS resource that runs your application\. The resource type appears under your application's name in the service map\. For example, `AWS::ElasticBeanstalk::Environment`\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the plugin that was loaded last to determine the origin\.

## Sampling Rules<a name="xray-sdk-go-configuration-sampling"></a>

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

On Lambda, you cannot modify the sampling rate\. If your function is called by an instrumented service, calls generated requests that were sampled by that service will be recorded by Lambda\. If active tracing is enabled and no tracing header is present, Lambda makes the sampling decision\.

To provide backup rules, point to the local sampling JSON file by using `NewCentralizedStrategyWithFilePath`\.

**Example main\.go – local sampling rule**  

```
s, _ := sampling.NewCentralizedStrategyWithFilePath("sampling.json") // path to local sampling json
xray.Configure(xray.Config{SamplingStrategy: s})
```

To use only local rules, point to the local sampling JSON file by using `NewLocalizedStrategyFromFilePath`\.

**Example main\.go – disable sampling**  

```
s, _ := sampling.NewLocalizedStrategyFromFilePath("sampling.json") // path to local sampling json
xray.Configure(xray.Config{SamplingStrategy: s})
```

## Logging<a name="xray-sdk-go-configuration-logging"></a>

You can change the log level and format with `xray.Configure`\.

**Example main\.go**  

```
func main() {
  http.Handle("/", xray.Handler(xray.NewFixedSegmentNamer("MyApp"), http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {

    xray.Configure(xray.Config{
        LogLevel: "warn",
        LogFormat: "[%Level] [%Time] %Msg%n"
    })

    w.Write([]byte("Hello!"))
  })))

  http.ListenAndServe(":8000", nil)
}
```

See [Using Configure](#xray-sdk-go-configuration-configure) for more information\.

## Environment Variables<a name="xray-sdk-go-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Go\. The SDK supports the following variables\.
+ `AWS_XRAY_TRACING_NAME` – Set the service name that the SDK uses for segments\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.

Environment variables override equivalent values set in code\.

## Using Configure<a name="xray-sdk-go-configuration-configure"></a>

You can also configure the X\-Ray SDK for Go using the `Configure` method\. `Configure` takes one argument, a `Config` object, with the following, optional fields\.

DaemonAddr  
This string specifies the host and port of the X\-Ray daemon listener\. If not specified, X\-Ray uses the value of the `AWS_XRAY_DAEMON_ADDRESS` environment variable\. If that value is not set, it uses "127\.0\.0\.1:2000"\.

ServiceVersion  
This string specifies the version of the service\. If not specified, X\-Ray uses the empty string \(""\)\.

SamplingStrategy  
This `SamplingStrategy` object specifies which of your application calls are traced\. If not specified, X\-Ray uses a `LocalizedSamplingStrategy`, which takes the strategy as defined in `xray/resources/DefaultSamplingRules.json`\.

StreamingStrategy  
This `StreamingStrategy` object specifies whether to stream a segment when **RequiresStreaming** returns **true**\. If not specified, X\-Ray uses a `DefaultStreamingStrategy` that streams a sampled segment if the number of subsegments is greater than 20\.

ExceptionFormattingStrategy  
This `ExceptionFormattingStrategy` object specifies how you want to handle various exceptions\. If not specified, X\-Ray uses a `DefaultExceptionFormattingStrategy` with an `XrayError` of type `error`, the error message, and stack trace\.

LogLevel  
This string specifies the default logging level for your application\. You can set this to "trace", "debug", "info", "warn" or "error"\. If not specified, X\-Ray uses "info"\.

LogFormat  
This string specifies the format of the log messages\. If not specified, X\-Ray uses "%Date\(2006\-01\-02T15:04:05Z07:00\) \[%Level\] %Msg%n"\.