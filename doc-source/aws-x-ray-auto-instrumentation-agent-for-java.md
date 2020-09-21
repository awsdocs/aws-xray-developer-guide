# AWS X\-Ray auto\-instrumentation agent for Java<a name="aws-x-ray-auto-instrumentation-agent-for-java"></a>

The AWS X\-Ray auto\-instrumentation agent for Java is a tracing solution that instruments your Java web applications with minimal development effort\. The agent enables tracing automatically for servlet\-based applications and all of the agent's downstream requests made with supported frameworks and libraries\. This includes downstream Apache HTTP requests, AWS SDK requests, and SQL queries made using a JDBC driver\. The agent automatically propagates X\-Ray context, including all active segments and sub\-segments, across threads\. All of the configurations and versatility of the X\-Ray SDK are still available with the Java agent, and suitable defaults were chosen to ensure the agent works out\-of\-box\.

The X\-Ray agent solution is best suited for servlet\-based, request\-response Java web application servers\. If your application uses an asynchronous framework, like Spring WebFlux, or is not well\-modeled as a request\-response service, you might want to consider manual instrumentation with the SDK instead\. 

The X\-Ray agent is built using the Distributed Systems Comprehension Toolkit, or DiSCo\. DiSCo is an open\-source framework for building Java agents that can be used in distributed systems\. While it is not necessary to understand DiSCo to use the X\-Ray agent, you can learn more about the project by visiting its [homepage on GitHub](https://github.com/awslabs/disco)\. The X\-Ray agent is also fully open sourced\. To view the source code, make contributions, or raise issues about the agent, visit its [repository on GitHub](https://github.com/aws/aws-xray-java-agent)\.

## Sample application<a name="XRayAutoInstrumentationAgent-SampleApp"></a>

The [eb\-java\-scorekeep](https://github.com/aws-samples/eb-java-scorekeep/tree/xray-agent) sample application was adapted to be instrumented with the X\-Ray agent\. This branch contains no servlet filter or recorder configuration, as these functions are done by the agent\. To run the application locally or using AWS resources, follow the steps in the sample application's readme file\. The instructions for using the sample app to generate X\-Ray traces are in the [sample app’s tutorial](xray-scorekeep.md)\.

## Getting started<a name="XRayAutoInstrumentationAgent-GettingStarted"></a>

To get started with the X\-Ray auto\-instrumentation Java agent in your own application, follow these steps\.

1. Run the X\-Ray daemon in your environment\. For more information see [X\-Ray daemon](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html)\.

1. Download the [latest distribution of the agent](https://github.com/aws/aws-xray-java-agent/releases/latest/download/xray-agent.zip)\. Unzip the archive and note its location in your file system\. Its contents should look like the following\.

   ```
   disco 
   ├── disco-java-agent.jar 
   └── disco-plugins 
       ├── aws-xray-agent-plugin.jar 
       ├── disco-java-agent-aws-plugin.jar 
       ├── disco-java-agent-sql-plugin.jar 
       └── disco-java-agent-web-plugin.jar
   ```

1. Modify the JVM arguments of your application to include the following, which enables the agent\. The process to modify JVM arguments varies depending on the tools and frameworks you use to launch your Java server, so consult the documentation of your server framework for specific guidance\.

   ```
   -javaagent:/<path-to-disco>/disco-java-agent.jar=pluginPath=/<path-to-disco>/disco-plugins
   ```

1. To specify the name of your application as it will appear on the X\-Ray console, set the `AWS_XRAY_TRACING_NAME` environment variable or the `com.amazonaws.xray.strategy.tracingName` system property\. If no name is provided, a default name is used\.

1. Restart your server or container\. Incoming requests and their downstream calls are now traced\. If you don’t see the expected results, see [Troubleshooting](#XRayAutoInstrumentationAgent-Troubleshooting)\.

## Configuration<a name="XRayAutoInstrumentationAgent-Configuration"></a>

The X\-Ray agent is configured by an external, user\-provided JSON file\. By default, this file is at the root of the user’s classpath \(for example, in their `resources` directory\) named `xray-agent.json`\. You can configure a custom location for the config file by setting the `com.amazonaws.xray.configFile` system property to the absolute filesystem path of your configuration file\.

An example configuration file is shown next\.

```
{     
    "serviceName": "XRayInstrumentedService", 
    "contextMissingStrategy": "LOG_ERROR", 
    "daemonAddress": "127.0.0.1:2000", 
    "tracingEnabled": true, 
    "samplingStrategy": "CENTRAL",     
    "traceIdInjectionPrefix": "prefix",     
    "samplingRulesManifest": "/path/to/manifest",     
    "awsServiceHandlerManifest": "/path/to/manifest",     
    "awsSdkVersion": 2,     
    "maxStackTraceLength": 50,     
    "streamingThreshold": 100,     
    "traceIdInjection": true,     
    "pluginsEnabled": true,     
    "collectSqlQueries": false 
}
```

### Configuration specification<a name="XRayAutoInstrumentationAgent-ConfigSpecs"></a>

The following table describes valid values for each property\. Property names are case sensitive, but their keys are not\. For properties that can be overridden by environment variables and system properties, the order of priority is always environment variable, then system property, and then configuration file\. See the [X\-Ray Docs](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html#xray-sdk-java-configuration-envvars) for information about properties that you can override\. All fields are optional\.


|  Property name  |  Type  |  Valid values  |  Description  |  Environment variable  |  System property  |  Default  | 
| --- | --- | --- | --- | --- | --- | --- | 
|  serviceName  |  String  |  Any string  |  The name of your instrumented service as it will appear in the X\-Ray console\.  |  AWS\_XRAY\_TRACING\_NAME  |  com\.amazonaws\.xray\.strategy\.tracingName  |  XRayInstrumentedService  | 
|  contextMissingStrategy  |  String  |  LOG\_ERROR, IGNORE\_ERROR  |  The action taken by the agent when it attempts to use the X\-Ray segment context but none is present\.  |  AWS\_XRAY\_CONTEXT\_MISSING  |  com\.amazonaws\.xray\.strategy\.contextMissingStrategy  |  LOG\_ERROR  | 
|  daemonAddress  |  String  |  Formatted IP address and port, or list of TCP and UDP address  |  The address the agent uses to communicate with the X\-Ray daemon\.  |  AWS\_XRAY\_DAEMON\_ADDRESS  |  com\.amazonaws\.xray\.emitter\.daemonAddress  |  127\.0\.0\.1:2000  | 
|  tracingEnabled  |  Boolean  |  True, False  |  Enables instrumentation by the X\-Ray agent\.  |  AWS\_XRAY\_TRACING\_ENABLED  |  com\.amazonaws\.xray\.tracingEnabled  |  TRUE  | 
|  samplingStrategy  |  String  |  CENTRAL, LOCAL, NONE, ALL  |  The sampling strategy used by the agent\. ALL captures all requests, NONE captures no requests\. See [sampling rules](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html#xray-sdk-java-configuration-sampling)\.  |  N/A  |  N/A  |  CENTRAL  | 
|  traceIdInjectionPrefix  |  String  |  Any string  |  Includes the provided prefix before injected trace IDs in logs\.  |  N/A  |  N/A  |  None \(empty string\)  | 
|  samplingRulesManifest  |  String  |  An absolute file path  |  The path to a custom sampling rules file to be used as the source of sampling rules for the local sampling strategy, or the fallback rules for the central strategy\.  |  N/A  |  N/A  |   [DefaultSamplingRules\.json](https://github.com/aws/aws-xray-sdk-java/blob/master/aws-xray-recorder-sdk-core/src/main/resources/com/amazonaws/xray/strategy/sampling/DefaultSamplingRules.json)   | 
|   `awsServiceHandlerManifest`   |  String  |  An absolute file path  |  The path to a custom parameter allow list, which captures additional information from AWS SDK clients\.  |  N/A  |  N/A  |   [DefaultOperationParameterWhitelist\.json](https://github.com/aws/aws-xray-sdk-java/blob/master/aws-xray-recorder-sdk-aws-sdk-v2/src/main/resources/com/amazonaws/xray/interceptors/DefaultOperationParameterWhitelist.json)   | 
|  awsSdkVersion  |  Integer  |  1, 2  |  Version of the [AWS SDK for Java](https://docs.aws.amazon.com/sdk-for-java/index.html) you’re using\. Ignored if `awsServiceHandlerManifest` is not also set\.  |  N/A  |  N/A  |  2  | 
|  maxStackTraceLength  |  Integer  |  Non\-negative integers  |  The maximum lines of a stack trace to record in a trace\.  |  N/A  |  N/A  |  50  | 
|  streamingThreshold  |  Integer  |  Non\-negative integers  |  After at least this many subsegments are closed, they are streamed to the daemon out\-of\-band to avoid chunks being too large\.  |  N/A  |  N/A  |  100  | 
|  traceIdInjection  |  Boolean  |  True, False  |  Enables X\-Ray trace ID injection into logs if the dependencies and configuration described in [logging config](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html#xray-sdk-java-configuration-logging) are also added\. Otherwise, does nothing\.  |  N/A  |  N/A  |  TRUE  | 
|  pluginsEnabled  |  Boolean  |  True, False  |  Enables plugins that record metadata about the AWS environments you’re operating in\. See [plugins](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html#xray-sdk-java-configuration-plugins)\.  |  N/A  |  N/A  |  TRUE  | 
|  collectSqlQueries  |  Boolean  |  True, False  |  Records SQL query strings in SQL subsegments on a best\-effort basis\.  |  N/A  |  N/A  |  FALSE  | 
|  contextPropagation  |  Boolean  |  True, False  |  Automatically propagates X\-Ray context between threads if true\. Otherwise uses Thread Local to store context and manual propagation across threads is required\.  |  N/A  |  N/A  |  TRUE  | 

### Manual instrumentation<a name="XRayAutoInstrumentationAgent-ManualInstrumentation"></a>

If you’d like to perform manual instrumentation in addition to the agent’s auto\-instrumentation, add the X\-Ray SDK as a dependency to your project\.

**Note**  
You must use the latest version of the X\-Ray SDK to perform manual instrumentation while also using the agent\. 

If you are working in a Maven project, add the following dependencies to your `pom.xml` file\. 

```
<dependencies> 
  <dependency> 
    <groupId>com.amazonaws</groupId> 
    <artifactId>aws-xray-recorder-sdk-core</artifactId> 
    <version>2.7.1</version> 
  </dependency> 
  </dependencies>
```

If you are working in a Gradle project, add the following dependencies to your `build.gradle` file\.

```
implementation 'com.amazonaws:aws-xray-recorder-sdk-core:2.7.1' 
```

You can add [ custom subsegments](xray-sdk-java-subsegments.md) as well as [annotations, metadata, and user IDs](xray-sdk-java-segment.md) while using the agent, just as you would with the normal SDK\. The agent automatically propagates context across threads, so no workarounds to propagate context should be necessary when working with multithreaded applications\.

## Troubleshooting<a name="XRayAutoInstrumentationAgent-Troubleshooting"></a>

Since the agent offers fully automatic instrumentation, it can be difficult to identify the root cause of a problem when you are experiencing issues\. If the X\-Ray agent is not working as expected for you, review the following problems and solutions\. The X\-Ray agent and SDK use Jakarta Commons Logging \(JCL\), so in order to see the logging output you’ll need to ensure that a bridge connecting JCL to your logging backend is on the classpath, for example `log4j-jcl` or `jcl-over-slf4j`\.

### Problem: I’ve enabled the Java agent on my application but don’t see anything on the X\-Ray console<a name="-problem-ive-enabled-the-java-agent-on-my-application-but-dont-see-anything-on-the-x-ray-console"></a>

 **Is the X\-Ray daemon running on the same machine?** 

If not, see the [X\-Ray daemon documentation](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html) to set it up\.

 **In your application logs, do you see a message like "Initializing the X\-Ray agent recorder"?** 

This message is logged at INFO level when your application starts, before it starts taking requests, if you have correctly added the agent to your application\. If this message is not there, then the Java agent is not running with your Java process\. Make sure you’ve followed all the setup steps correctly with no typos\.

 **In your application logs, do you see several error messages saying something like "Suppressing AWS X\-Ray context missing exception"?** 

These errors occur because the agent is trying to instrument downstream requests, like AWS SDK requests or SQL queries, but the agent was unable to automatically create a segment\. If you see many of these errors, the agent might not be the best tool for your use case and you might want to consider manual instrumentation with the X\-Ray SDK instead\. Alternatively, you can enable X\-Ray SDK [debug logs](https://docs.aws.amazon.com/xray/latest/devguide/ray-sdk-java-configuration.html#xray-sdk-java-configuration-logging) to see the stack trace of where the context\-missing exceptions are occurring\. You can wrap these portions of your code with custom segments, which should resolve these errors\. For an example of wrapping downstream requests with custom segments, see the sample code in [instrumenting startup code](https://docs.aws.amazon.com/xray/latest/devguide/scorekeep-startup.html)\.

### Problem: Some of the segments I expect do not appear on the X\-Ray console<a name="-problem-some-but-not-all-the-segments-im-expecting-appear-on-the-x-ray-console"></a>

 **Does your application use multithreading?**

 If some segments that you expect to be created are not appearing in your console, background threads in your application might be the cause\. If your application performs tasks using background threads that are “fire and forget,” like making a one\-off call to a Lambda function with the AWS SDK, or polling some HTTP endpoint periodically, that may confuse the agent while it is propagating context across threads\. To verify this is your problem, enable X\-Ray SDK debug logs and check for messages like: *Not emitting segment named <NAME > as it parents in\-progress subsegments*\. To work around this, you can try joining the background threads before your server returns to ensure all the work done in them is recorded\. Or, you can set the agent’s `contextPropagation` configuration to `false` to disable context propagation in background threads\. If you do this, you’ll have to manually instrument those threads with custom segments or ignore the context missing exceptions they produce\. 

**Have you set up sampling rules?** 

 If there are seemingly random or unexpected segments appearing on the X\-Ray console, or the segments you expect to be on the console aren’t, you might be experiencing a sampling issue\. The X\-Ray agent applies centralized sampling to all segments it creates, using the rules from the X\-Ray console\. The default rule is 1 segment per second, plus 5% of segments afterward, are sampled\. This means segments that are created rapidly with the agent might not be sampled\. To resolve this, you should create custom sampling rules on the X\-Ray console that appropriately sample the desired segments\. For more information, see [sampling](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html)\. 