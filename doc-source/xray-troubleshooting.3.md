# Troubleshooting AWS X\-Ray<a name="xray-troubleshooting"></a>

This topic lists common errors and issues that you might encounter when using the X\-Ray API, console, or SDKs\. If you find an issue that is not listed here, you can use the **Feedback** button on this page to report it\.

**Topics**
+ [X\-Ray SDK for Java](#troubleshooting-java)
+ [X\-Ray SDK for Node\.js](#troubleshooting-nodejs)
+ [The X\-Ray daemon](#troubleshooting-daemon)

## X\-Ray SDK for Java<a name="troubleshooting-java"></a>

**Error:** *Exception in thread "Thread\-1" com\.amazonaws\.xray\.exceptions\.SegmentNotFoundException: Failed to begin subsegment named 'AmazonSNS': segment cannot be found\. *

This error indicates that the X\-Ray SDK attempted to record an outgoing call to AWS, but couldn't find an open segment\. This can occur in the following situations:
+ **A servlet filter is not configured** – The X\-Ray SDK creates segments for incoming requests with a filter named `AWSXRayServletFilter`\. [Configure a servlet filter](xray-sdk-java-filters.md) to instrument incoming requests\.
+ **You're using instrumented clients outside of servlet code** – If you use an instrumented client to make calls in startup code or other code that doesn't run in response to an incoming request, you must create a segment manually\. See [Instrumenting startup code](scorekeep-startup.md) for examples\.
+ **You're using instrumented clients in worker threads** – When you create a new thread, the X\-Ray recorder loses its reference to the open segment\. You can use the [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html#getTraceEntity--](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html#getTraceEntity--) and [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html#setTraceEntity--](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRayRecorder.html#setTraceEntity--) methods to get a reference to the current segment or subsegment \([https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Entity.html](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Entity.html)\), and pass it back to the recorder inside of the thread\. See [Using instrumented clients in worker threads](scorekeep-workerthreads.md) for an example\.

## X\-Ray SDK for Node\.js<a name="troubleshooting-nodejs"></a>

**Issue:** *CLS does not work with Sequelize*

Pass the X\-Ray SDK for Node\.js namespace to Sequelize with the `cls` method\.

```
var AWSXRay = require('aws-xray-sdk');
const Sequelize = require('sequelize');
Sequelize.cls = AWSXRay.getNamespace();
const sequelize = new Sequelize('database', 'username', 'password');
```

**Issue:** *CLS does not work with Bluebird*

Use `cls-bluebird` to get Bluebird working with CLS\.

```
var AWSXRay = require('aws-xray-sdk');
var Promise = require('bluebird');
var clsBluebird = require('cls-bluebird');
clsBluebird(AWSXRay.getNamespace());
```

## The X\-Ray daemon<a name="troubleshooting-daemon"></a>

**Issue:** *The daemon is using the wrong credentials*

The daemon uses the AWS SDK to load credentials\. If you use multiple methods of providing credentials, the method with the highest precedence is used\. See [Running the daemon](xray-daemon.md#xray-daemon-running) for more information\.