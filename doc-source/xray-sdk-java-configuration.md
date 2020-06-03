# Configuring the X\-Ray SDK for Java<a name="xray-sdk-java-configuration"></a>

The X\-Ray SDK for Java includes a class named `AWSXRay` that provides the global recorder\. This is a `TracingHandler` that you can use to instrument your code\. You can configure the global recorder to customize the `AWSXRayServletFilter` that creates segments for incoming HTTP calls\.

**Topics**
+ [Service plugins](#xray-sdk-java-configuration-plugins)
+ [Sampling rules](#xray-sdk-java-configuration-sampling)
+ [Logging](#xray-sdk-java-configuration-logging)
+ [Segment listeners](#xray-sdk-java-configuration-listeners)
+ [Environment variables](#xray-sdk-java-configuration-envvars)
+ [System properties](#xray-sdk-java-configuration-sysprops)

## Service plugins<a name="xray-sdk-java-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**
+ Amazon EC2 – `EC2Plugin` adds the instance ID, Availability Zone, and the CloudWatch Logs Group\.
+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ECSPlugin` adds the container ID\.
+ Amazon EKS – `EKSPlugin` adds the container ID, cluster name, pod ID, and the CloudWatch Logs Group\.

![\[Segment resource data with Amazon EC2 and Elastic Beanstalk plugins.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-segment-resources.png)

To use a plugin, call `withPlugin` on your `AWSXRayRecorderBuilder`\.

**Example src/main/java/scorekeep/WebConfig\.java \- recorder**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.AWSXRayRecorderBuilder](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorderBuilder.html);
import [com\.amazonaws\.xray\.plugins\.EC2Plugin](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/EC2Plugin.html);
import [com\.amazonaws\.xray\.plugins\.ElasticBeanstalkPlugin](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/ElasticBeanstalkPlugin.html);
import [com\.amazonaws\.xray\.strategy\.sampling\.LocalizedSamplingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/sampling/LocalizedSamplingStrategy.html);

@Configuration
public class WebConfig {
...
  static {
    AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder.standard().withPlugin(new EC2Plugin()).withPlugin(new ElasticBeanstalkPlugin());

    URL ruleFile = WebConfig.class.getResource("/sampling-rules.json");
    builder.withSamplingStrategy(new LocalizedSamplingStrategy(ruleFile));

    AWSXRay.setGlobalRecorder(builder.build());
  }
}
```

The SDK also uses plugin settings to set the `origin` field on the segment\. This indicates the type of AWS resource that runs your application\. The resource type appears under your application's name in the service map\. For example, `AWS::ElasticBeanstalk::Environment`\.

![\[Service node with resource type.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the following resolution order to determine the origin: ElasticBeanstalk > EKS > ECS > EC2\.

## Sampling rules<a name="xray-sdk-java-configuration-sampling"></a>

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

To provide backup rules in Spring, configure the global recorder with a `CentralizedSamplingStrategy` in a configuration class\.

**Example src/main/java/myapp/WebConfig\.java \- recorder configuration**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.AWSXRayRecorderBuilder](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorderBuilder.html);
import [com\.amazonaws\.xray\.javax\.servlet\.AWSXRayServletFilter](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html);
import [com\.amazonaws\.xray\.plugins\.EC2Plugin](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/EC2Plugin.html);
import [com\.amazonaws\.xray\.strategy\.sampling\.LocalizedSamplingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/sampling/LocalizedSamplingStrategy.html);

@Configuration
public class WebConfig {

  static {
  AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder.standard().withPlugin(new EC2Plugin());

  URL ruleFile = WebConfig.class.getResource("file://sampling-rules.json");
  builder.withSamplingStrategy(new CentralizedSamplingStrategy(ruleFile));

  AWSXRay.setGlobalRecorder(builder.build());
}
```

For Tomcat, add a listener that extends `ServletContextListener` and register the listener in the deployment descriptor\.

**Example src/com/myapp/web/Startup\.java**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.AWSXRayRecorderBuilder](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorderBuilder.html);
import [com\.amazonaws\.xray\.plugins\.EC2Plugin](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/EC2Plugin.html);
import [com\.amazonaws\.xray\.strategy\.sampling\.LocalizedSamplingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/sampling/LocalizedSamplingStrategy.html);

import java.net.URL;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

public class Startup implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder.standard().withPlugin(new EC2Plugin());

        URL ruleFile = Startup.class.getResource("/sampling-rules.json");
        builder.withSamplingStrategy(new CentralizedSamplingStrategy(ruleFile));

        AWSXRay.setGlobalRecorder(builder.build());
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) { }
}
```

**Example WEB\-INF/web\.xml**  

```
...
  <listener>
    <listener-class>com.myapp.web.Startup</listener-class>
  </listener>
```

To use local rules only, replace the `CentralizedSamplingStrategy` with a `LocalizedSamplingStrategy`\.

```
builder.withSamplingStrategy(new LocalizedSamplingStrategy(ruleFile));
```

## Logging<a name="xray-sdk-java-configuration-logging"></a>

By default, the SDK outputs `SEVERE`\-level and `ERROR`\-level messages to your application logs\. You can enable debug\-level logging on the SDK to output more detailed logs to your application log file\.

**Example application\.properties**  
Set the logging level with the `logging.level.com.amazonaws.xray` property\.  

```
logging.level.com.amazonaws.xray = DEBUG
```

Use debug logs to identify issues, such as unclosed subsegments, when you [generate subsegments manually](xray-sdk-java-subsegments.md)\.

### Trace ID injection into logs<a name="xray-sdk-java-configuration-logging-id-injection"></a>

To expose the current fully qualified trace ID to your log statements, you can inject the ID into the mapped diagnostic context \(MDC\)\. Using the `SegmentListener` interface, methods are called from the X\-Ray recorder during segment lifecycle events\. When a segment or subsegment begins, the qualified trace ID is injected into the MDC with the key `AWS-XRAY-TRACE-ID`\. When that segment ends, the key is removed from the MDC\. This exposes the trace ID to the logging library in use\. When a subsegment ends, its parent ID is injected into the MDC\.

**Example fully qualified trace ID**  
The fully qualified ID is represented as `TraceID@EntityID`  

```
1-5df42873-011e96598b447dfca814c156@541b3365be3dafc3
```

This feature works with Java applications instrumented with the AWS X\-Ray SDK for Java, and supports the following logging configurations:
+ SLF4J front\-end API with Logback backend
+ SLF4J front\-end API with Log4J2 backend
+ Log4J2 front\-end API with Log4J2 backend

See the following tabs for the needs of each front end and each backend\.

------
#### [ SLF4J Frontend ]

1. Add the following Maven dependency to your project\.

   ```
   <dependency>
       <groupId>com.amazonaws</groupId>
       <artifactId>aws-xray-recorder-sdk-slf4j</artifactId>
       <version>2.4.0</version>
   </dependency>
   ```

1. Include the `withSegmentListener` method when building the `AWSXRayRecorder`\. This adds a `SegmentListener` class, which automatically injects new trace IDs into the SLF4J MDC\.

   The `SegmentListener` takes an optional string as a parameter to configure the log statement prefix\. The prefix can be configured in the following ways:
   + **None** – Uses the default `AWS-XRAY-TRACE-ID` prefix\.
   + **Empty** – Uses an empty string \(e\.g\. `""`\)\.
   + **Custom** – Uses a custom prefix as defined in the string\.  
**Example `AWSXRayRecorderBuilder` statement**  

   ```
   AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder
           .standard().withSegmentListener(new SLF4JSegmentListener("CUSTOM-PREFIX"));
   ```

------
#### [ Log4J2 front end ]

1. Add the following Maven dependency to your project\.

   ```
   <dependency>
       <groupId>com.amazonaws</groupId>
       <artifactId>aws-xray-recorder-sdk-log4j</artifactId>
       <version>2.4.0</version>
   </dependency>
   ```

1. Include the `withSegmentListener` method when building the `AWSXRayRecorder`\. This will add a `SegmentListener` class, which automatically injects new fully qualified trace IDs into the SLF4J MDC\.

   The `SegmentListener` takes an optional string as a parameter to configure the log statement prefix\. The prefix can be configured in the following ways:
   + **None** – Uses the default `AWS-XRAY-TRACE-ID` prefix\.
   + **Empty** – Uses an empty string \(e\.g\. `""`\) and removes the prefix\.
   + **Custom** – Uses the custom prefix defined in the string\.  
**Example `AWSXRayRecorderBuilder` statement**  

   ```
   AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder
           .standard().withSegmentListener(new Log4JSegmentListener("CUSTOM-PREFIX"));
   ```

------
#### [ Logback backend ]

To insert the trace ID into your log events, you must modify the logger's `PatternLayout`, which formats each logging statement\.

1. Find where the `patternLayout` is configured\. You can do this programmatically, or through an XML configuration file\. To learn more, see [Logback configuration](http://logback.qos.ch/manual/configuration.html)\.

1. Insert `%X{AWS-XRAY-TRACE-ID}` anywhere in the `patternLayout` to insert the trace ID in future logging statements\. `%X{}` indicates that you are retrieving a value with the provided key from the MDC\. To learn more about PatternLayouts in Logback, see [PatternLayout](https://logback.qos.ch/manual/layouts.html#ClassicPatternLayout)\.

------
#### [ Log4J2 backend ]

1. Find where the `patternLayout` is configured\. You can do this programmatically, or through a configuration file written in XML, JSON, YAML, or properties format\. 

   To learn more about configuring Log4J2 through a configuration file, see [Configuration](https://logging.apache.org/log4j/2.x/manual/configuration.html)\. 

   To learn more about configuring Log4J2 programmatically, see [Programmatic Configuration](https://logging.apache.org/log4j/2.x/manual/customconfig.html)\. 

1. Insert `%X{AWS-XRAY-TRACE-ID}` anywhere in the `PatternLayout` to insert the trace ID in future logging statements\. `%X{}` indicates that you are retrieving a value with the provided key from the MDC\. To learn more about PatternLayouts in Log4J2, see [Pattern Layout](https://logging.apache.org/log4j/2.x/manual/layouts.html#Pattern_Layout)\.

------

**Trace ID Injection Example**  
The following shows a `PatternLayout` string modified to include the trace ID\. The trace ID is printed after the thread name \(`%t`\) and before the log level \(`%-5p`\)\.

**Example `PatternLayout` With ID injection**  

```
%d{HH:mm:ss.SSS} [%t] %X{AWS-XRAY-TRACE-ID} %-5p %m%n
```

AWS X\-Ray automatically prints the key and the trace ID in the log statement for easy parsing\. The following shows a log statement using the modified `PatternLayout`\.

**Example Log statement with ID injection**  

```
2019-09-10 18:58:30.844 [nio-5000-exec-4]  AWS-XRAY-TRACE-ID: 1-5d77f256-19f12e4eaa02e3f76c78f46a@1ce7df03252d99e1 WARN 1 - Your logging message here
```

 The logging message itself is housed in the pattern `%m` and is set when calling the logger\.

## Segment listeners<a name="xray-sdk-java-configuration-listeners"></a>

Segement listeners are an interface to intercept lifecycle events such as the beginning and ending of segments produced by the `AWSXRayRecorder`\. Implementation of a segment listener event function might be to add the same annotation to all subsegments when they are created with [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#onBeginSubsegment-com.amazonaws.xray.entities.Subsegment-](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#onBeginSubsegment-com.amazonaws.xray.entities.Subsegment-), log a message after each segment is sent to the daemon using [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#afterEndSegment-com.amazonaws.xray.entities.Segment-](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#afterEndSegment-com.amazonaws.xray.entities.Segment-), or to record queries sent by the SQL interceptors using [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#beforeEndSubsegment-com.amazonaws.xray.entities.Subsegment-](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#beforeEndSubsegment-com.amazonaws.xray.entities.Subsegment-) to verify if the subsegment represents an SQL query, adding additional metadata if so\.

To see the full list of `SegmentListener` functions, visit the documentation for the [AWS X\-Ray Recorder SDK for Java API](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html)\.

The following example shows how to add a consistent annotation to all subsegments on creation with [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#onBeginSubsegment-com.amazonaws.xray.entities.Subsegment-](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#onBeginSubsegment-com.amazonaws.xray.entities.Subsegment-) and to print a log message at the end of each segment with [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#afterEndSegment-com.amazonaws.xray.entities.Segment-](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/listeners/SegmentListener.html#afterEndSegment-com.amazonaws.xray.entities.Segment-)\. 

**Example MySegmentListener\.java**  

```
import com.amazonaws.xray.entities.Segment;
import com.amazonaws.xray.entities.Subsegment;
import com.amazonaws.xray.listeners.SegmentListener;

public class MySegmentListener implements SegmentListener {
    .....
    
    @Override
    public void onBeginSubsegment(Subsegment subsegment) {
        subsegment.putAnnotation("annotationKey", "annotationValue");
    }
    
    @Override
    public void afterEndSegment(Segment segment) {
        // Be mindful not to mutate the segment
        logger.info("Segment with ID " + segment.getId());
    }
}
```

This custom segment listener is then referenced when building the `AWSXRayRecorder`\.

**Example AWSXRayRecorderBuilder statement**  

```
AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder
        .standard().withSegmentListener(new MySegmentListener());
```

## Environment variables<a name="xray-sdk-java-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Java\. The SDK supports the following variables\.
+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the service name that you set on the servlet filter's [segment naming strategy](xray-sdk-java-filters.md#xray-sdk-java-filters-naming)\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK uses `127.0.0.1:2000` for both trace data \(UDP\) and sampling \(TCP\)\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.

**Format**
  + **Same port** – `address:port`
  + **Different ports** – `tcp:address:port udp:address:port`
+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**
  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.
  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.

Environment variables override equivalent [system properties](#xray-sdk-java-configuration-sysprops) and values set in code\.

## System properties<a name="xray-sdk-java-configuration-sysprops"></a>

You can use system properties as a JVM\-specific alternative to [environment variables](#xray-sdk-java-configuration-envvars)\. The SDK supports the following properties:
+ `com.amazonaws.xray.strategy.tracingName` – Equivalent to `AWS_XRAY_TRACING_NAME`\.
+ `com.amazonaws.xray.emitters.daemonAddress` – Equivalent to `AWS_XRAY_DAEMON_ADDRESS`\.
+ `com.amazonaws.xray.strategy.contextMissingStrategy` – Equivalent to `AWS_XRAY_CONTEXT_MISSING`\.

If both a system property and the equivalent environment variable are set, the environment variable value is used\. Either method overrides values set in code\.