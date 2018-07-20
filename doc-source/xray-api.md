# AWS X\-Ray API<a name="xray-api"></a>

The X\-Ray API provides access to all X\-Ray functionality through the AWS SDK, AWS Command Line Interface, or directly over HTTPS\. The [X\-Ray API Reference](http://docs.aws.amazon.com//xray/latest/api/Welcome.html) documents input parameters each API action, and the fields and data types that they return\.

You can use the AWS SDK to develop programs that use the X\-Ray API\. The X\-Ray console and X\-Ray daemon both use the AWS SDK to communicate with X\-Ray\. The AWS SDK for each language has a reference document for classes and methods that map to X\-Ray API actions and types\.

**AWS SDK References**
+ **Java** – [AWS SDK for Java](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/xray/package-summary.html)
+ **JavaScript** – [AWS SDK for JavaScript](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/XRay.html)
+ **\.NET** – [AWS SDK for \.NET](http://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/XRay/NXRay.html)
+ **Ruby** – [AWS SDK for Ruby](http://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/XRay.html)
+ **Go** – [AWS SDK for Go](http://docs.aws.amazon.com/sdk-for-go/api/service/xray/)
+ **PHP** – [AWS SDK for PHP](http://docs.aws.amazon.com/aws-sdk-php/v3/api/namespace-Aws.XRay.html)
+ **Python** – [AWS SDK for Python \(Boto\)](http://boto3.readthedocs.org/en/latest/reference/services/xray.html)

The AWS Command Line Interface is a command line tool that uses the SDK for Python to call AWS APIs\. When you are first learning an AWS API, the AWS CLI provides an easy way to explore the available parameters and view the service output in JSON or text form\.

See [the AWS CLI Command Reference](http://docs.aws.amazon.com/cli/latest/reference/xray) for details on `aws xray` subcommands\.

**Topics**
+ [Using the AWS X\-Ray API with the AWS CLI](xray-api-tutorial.md)
+ [Sending Trace Data to AWS X\-Ray](xray-api-sendingdata.md)
+ [Getting Data from AWS X\-Ray](xray-api-gettingdata.md)
+ [Configuring AWS X\-Ray Encryption Settings](xray-api-configuration.md)
+ [Logging X\-Ray API Calls with AWS CloudTrail](xray-api-cloudtrail.md)
+ [Tracking X\-Ray Encryption Configuration Changes with AWS Config](xray-api-config.md)
+ [AWS X\-Ray Segment Documents](xray-api-segmentdocuments.md)