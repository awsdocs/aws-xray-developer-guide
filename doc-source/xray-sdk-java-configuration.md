# Configuring the X\-Ray SDK for Java<a name="xray-sdk-java-configuration"></a>

The X\-Ray SDK for Java provides a class named `AWSXRay` that provides the global recorder, a `TracingHandler` that you can use to instrument your code\. You can configure the global recorder to customize the `AWSXRayServletFilter` that creates segments for incoming HTTP calls\.

**Topics**
+ [Service Plugins](#xray-sdk-java-configuration-plugins)
+ [Sampling Rules](#xray-sdk-java-configuration-sampling)
+ [Logging](#xray-sdk-java-configuration-logging)
+ [Environment Variables](#xray-sdk-java-configuration-envvars)
+ [System Properties](#xray-sdk-java-configuration-sysprops)

## Service Plugins<a name="xray-sdk-java-configuration-plugins"></a>

Use `plugins` to record information about the service hosting your application\.

**Plugins**
+ Amazon EC2 – `EC2Plugin` adds the instance ID and Availability Zone\.
+ Elastic Beanstalk – `ElasticBeanstalkPlugin` adds the environment name, version label, and deployment ID\.
+ Amazon ECS – `ECSPlugin` adds the container ID\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-segment-resources.png)

To use a plugin, call `withPlugin` on your `AWSXRayRecorderBuilder`\.

**Example src/main/java/scorekeep/WebConfig\.java \- Recorder**  

```
import [com\.amazonaws\.xray\.AWSXRay](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.AWSXRayRecorderBuilder](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorderBuilder.html);
import [com\.amazonaws\.xray\.plugins\.EC2Plugin](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/EC2Plugin.html);
import [com\.amazonaws\.xray\.plugins\.ElasticBeanstalkPlugin](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/ElasticBeanstalkPlugin.html);
import [com\.amazonaws\.xray\.strategy\.sampling\.LocalizedSamplingStrategy](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/sampling/LocalizedSamplingStrategy.html);

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

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode.png)

When you use multiple plugins, the SDK uses the plugin that was loaded last to determine the origin\.

## Sampling Rules<a name="xray-sdk-java-configuration-sampling"></a>

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

For Spring, configure the global recorder in a configuration class\.

**Example src/main/java/myapp/WebConfig\.java \- Recorder Configuration**  

```
import [com\.amazonaws\.xray\.AWSXRay](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.AWSXRayRecorderBuilder](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorderBuilder.html);
import [com\.amazonaws\.xray\.javax\.servlet\.AWSXRayServletFilter](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html);
import [com\.amazonaws\.xray\.plugins\.EC2Plugin](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/EC2Plugin.html);
import [com\.amazonaws\.xray\.strategy\.sampling\.LocalizedSamplingStrategy](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/sampling/LocalizedSamplingStrategy.html);

@Configuration
public class WebConfig {

  static {
  AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder.standard().withPlugin(new EC2Plugin());

  URL ruleFile = WebConfig.class.getResource("file://sampling-rules.json");
  builder.withSamplingStrategy(new LocalizedSamplingStrategy(ruleFile));

  AWSXRay.setGlobalRecorder(builder.build());
}
```

For Tomcat, add a listener that extends `ServletContextListener`\.

**Example src/com/myapp/web/Startup\.java**  

```
import [com\.amazonaws\.xray\.AWSXRay](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.AWSXRayRecorderBuilder](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorderBuilder.html);
import [com\.amazonaws\.xray\.plugins\.EC2Plugin](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/plugins/EC2Plugin.html);
import [com\.amazonaws\.xray\.strategy\.sampling\.LocalizedSamplingStrategy](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/sampling/LocalizedSamplingStrategy.html);

import java.net.URL;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

public class Startup implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder.standard().withPlugin(new EC2Plugin());

        URL ruleFile = Startup.class.getResource("/sampling-rules.json");
        builder.withSamplingStrategy(new LocalizedSamplingStrategy(ruleFile));

        AWSXRay.setGlobalRecorder(builder.build());
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) { }
}
```

Register the listener in the deployment descriptor\.

**Example WEB\-INF/web\.xml**  

```
...
  <listener>
    <listener-class>com.myapp.web.Startup</listener-class>
  </listener>
```

## Logging<a name="xray-sdk-java-configuration-logging"></a>

By default, the SDK outputs `SEVERE` level and `ERROR` level messages to your application logs\. You can enable debug\-level logging on the SDK to output more detailed logs to your application log file\.

**Example application\.properties**  
Set the logging level with the `logging.level.com.amazonaws.xray` property\.  

```
logging.level.com.amazonaws.xray = DEBUG
```

Use debug logs to identify issues, such as unclosed subsegments, when you [generate subsegments manually](xray-sdk-java-subsegments.md)\.

## Environment Variables<a name="xray-sdk-java-configuration-envvars"></a>

You can use environment variables to configure the X\-Ray SDK for Java\. The SDK supports the following variables\.
+ `AWS_XRAY_TRACING_NAME` – Set a service name that the SDK uses for segments\. Overrides the service name that you set on the servlet filter's [segment naming strategy](xray-sdk-java-filters.md#xray-sdk-java-filters-naming)\.
+ `AWS_XRAY_DAEMON_ADDRESS` – Set the host and port of the X\-Ray daemon listener\. By default, the SDK sends trace data to `127.0.0.1:2000`\. Use this variable if you have configured the daemon to [listen on a different port](xray-daemon-configuration.md) or if it is running on a different host\.
+ `AWS_XRAY_CONTEXT_MISSING` – Set to `LOG_ERROR` to avoid throwing exceptions when your instrumented code attempts to record data when no segment is open\.

**Valid Values**
  + `RUNTIME_ERROR` – Throw a runtime exception \(default\)\.
  + `LOG_ERROR` – Log an error and continue\.

  Errors related to missing segments or subsegments can occur when you attempt to use an instrumented client in startup code that runs when no request is open, or in code that spawns a new thread\.

Environment variables override equivalent [system properties](#xray-sdk-java-configuration-sysprops) and values set in code\.

## System Properties<a name="xray-sdk-java-configuration-sysprops"></a>

You can use system properties as a JVM\-specific alternative to [environment variables](#xray-sdk-java-configuration-envvars)\. The SDK supports the following properties\.
+ `com.amazonaws.xray.strategy.tracingName` – Equivalent to `AWS_XRAY_TRACING_NAME`\.
+ `com.amazonaws.xray.emitters.daemonAddress` – Equivalent to `AWS_XRAY_DAEMON_ADDRESS`\.
+ `com.amazonaws.xray.strategy.contextMissingStrategy` – Equivalent to `AWS_XRAY_CONTEXT_MISSING`\.

If both a system property and the equivalent environment variable are set, the environment variable values is used\. Either method overrides values set in code\.