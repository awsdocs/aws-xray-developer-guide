# Configuring the X\-Ray SDK for Go<a name="xray-sdk-go-configuration"></a>

You can specify the configuration for for X\-Ray SDK for Go through environment variables, by calling `Configure` with a `Config` object, or by assuming default values\. Environment variables take precedence over `Config` values, which take precedence over any default value\.

**Topics**
+ [Service plugins](#xray-sdk-go-configuration-plugins)
+ [Sampling rules](#xray-sdk-go-configuration-sampling)
+ [Logging](#xray-sdk-go-configuration-logging)
+ [Environment variables](#xray-sdk-go-configuration-envvars)
+ [Using configure](#xray-sdk-go-configuration-configure)

## Service plugins<a name="xray-sdk-go-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**
+ Amazon EC2 – `EC2Plugin` adds the instance ID, Availability Zone, and the CloudWatch Logs Group\.
+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ECSPlugin` adds the container ID\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-segment-resources-go.png)

To use a plugin, import one of the following packages\.

```
"github.com/aws/aws-xray-sdk-go/awsplugins/ec2"
"github.com/aws/aws-xray-sdk-go/awsplugins/ecs"
"github.com/aws/aws-xray-sdk-go/awsplugins/beanstalk"
```

Each plugin has an explicit `Init()` function call that loads the plugin\.

**Example ec2\.Init\(\)**  

```
import (
	"os"

	"github.com/aws/aws-xray-sdk-go/awsplugins/ec2"
	"github.com/aws/aws-xray-sdk-go/xray"
	)

	func init() {
		// conditionally load plugin
		if os.Getenv("ENVIRONMENT") == "production" {
		ec2.Init()
	}

	xray.Configure(xray.Config{
		ServiceVersion: "1.2.3",

	})
}
```

The SDK also uses plugin settings to set the `origin` field on the segment\. This indicates the type of AWS resource that runs your application\. The resource type appears under your application's name in the service map\. For example, `AWS::ElasticBeanstalk::Environment`\.

![\[Service node with resource type.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the following resolution order to determine the origin: ElasticBeanstalk > EKS > ECS > EC2\.

## Sampling rules<a name="xray-sdk-go-configuration-sampling"></a>

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

To provide backup rules, point to the local sampling JSON file by using `NewCentralizedStrategyWithFilePath`\.

**Example main\.go – Local sampling rule**  

```
s, _ := sampling.NewCentralizedStrategyWithFilePath("sampling.json") // path to local sampling json
xray.Configure(xray.Config{SamplingStrategy: s})
```

To use only local rules, point to the local sampling JSON file by using `NewLocalizedStrategyFromFilePath`\.

**Example main\.go – Disable sampling**  

```
s, _ := sampling.NewLocalizedStrategyFromFilePath("sampling.json") // path to local sampling json
xray.Configure(xray.Config{SamplingStrategy: s})
```

## Logging<a name="xray-sdk-go-configuration-logging"></a>

**Note**  
The `xray.Config{}` fields `LogLevel` and `LogFormat` are deprecated starting with version 1\.0\.0\-rc\.10\.

X\-Ray uses the following interface for logging\. The default logger writes to `stdout` at `LogLevelInfo` and above\.

```
type Logger interface {
	Log(level LogLevel, msg fmt.Stringer)
}

const (
	LogLevelDebug LogLevel = iota + 1
	LogLevelInfo
	LogLevelWarn
	LogLevelError
)
```

**Example write to `io.Writer`**  

```
xray.SetLogger(xraylog.NewDefaultLogger(os.Stderr, xraylog.LogLevelError))
```

## Environment variables<a name="xray-sdk-go-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Go\. The SDK supports the following variables\.
+ `AWS_XRAY_TRACING_NAME` – Set the service name that the SDK uses for segments\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.
+ `AWS_XRAY_CONTEXT_MISSING` – Set the value to determine how the SDK handles missing context errors\. Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in the startup code when no request is open, or in code that spawns a new thread\. 
  + `RUNTIME_ERROR` – By default, the SDK is set to throw a runtime exception\.
  + `LOG_ERROR` – Set to log an error and continue\.

Environment variables override equivalent values set in code\.

## Using configure<a name="xray-sdk-go-configuration-configure"></a>

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