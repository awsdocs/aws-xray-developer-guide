# Getting started with AWS X\-Ray<a name="xray-gettingstarted"></a>

To get started with AWS X\-Ray, launch a sample app in Elastic Beanstalk that is already [instrumented](xray-sdk-java.md) to generate trace data\. In a few minutes, you can launch the sample app, generate traffic, send segments to X\-Ray, and view a service graph and traces in the AWS Management Console\.

This tutorial uses a [sample Java application](xray-scorekeep.md) to generate segments and send them to X\-Ray\. The application uses the Spring framework to implement a JSON web API and the AWS SDK for Java to persist data to Amazon DynamoDB\. A servlet filter in the application instruments all incoming requests served by the application, and a request handler on the AWS SDK client instruments downstream calls to DynamoDB\.

![\[Scorekeep sample application flow\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-flow.png)

You use the X\-Ray console to view the connections among client, server, and DynamoDB in a service map\. The service map is a visual representation of the services that make up your web application, generated from the trace data that it generates by serving requests\.

![\[Viewing the connections among client, server, and DynamoDB in a service map\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-after.png)

With the X\-Ray SDK for Java, you can trace all of your application's primary and downstream AWS resources by making two configuration changes:
+ Add the X\-Ray SDK for Java's tracing filter to your servlet configuration in a `WebConfig` class or `web.xml` file\.
+ Take the X\-Ray SDK for Java's submodules as build dependencies in your Maven or Gradle build configuration\.

You can also access the raw service map and trace data by using the AWS CLI to call the X\-Ray API\. The service map and trace data are JSON that you can query to ensure that your application is sending data, or to check specific fields as part of your test automation\.

**Topics**
+ [Prerequisites](#xray-gettingstarted-prereqs)
+ [Deploy to Elastic Beanstalk and generate trace data](#xray-gettingstarted-deploy)
+ [View the service map in the X\-Ray console](#xray-gettingstarted-console)
+ [Configuring Amazon SNS notifications](#xray-gettingstarted-notifications)
+ [Explore the sample application](#xray-gettingstarted-sample)
+ [Optional: Least privilege policy](#xray-gettingstarted-security)
+ [Clean up](#xray-gettingstarted-cleanup)
+ [Next steps](#xray-gettingstarted-nextsteps)

## Prerequisites<a name="xray-gettingstarted-prereqs"></a>

This tutorial uses Elastic Beanstalk to create and configure the resources that run the sample application and X\-Ray daemon\. If you use an IAM user with limited permissions, add the [Elastic Beanstalk managed user policy](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.iam.managed-policies.html) to grant your IAM user permission to use Elastic Beanstalk, and the `AWSXrayReadOnlyAccess` managed policy for permission to read the service map and traces in the X\-Ray console\.

Create an Elastic Beanstalk environment for the sample application\. If you haven't used Elastic Beanstalk before, this will also create a service role and instance profile for your application\. A default VPC must exist in the region you're going to deploy to or Elastic Beanstalk will fail to deploy the sample application\.

**To create an Elastic Beanstalk environment**

1. Open the Elastic Beanstalk Management Console with this preconfigured link: [https://console\.aws\.amazon\.com/elasticbeanstalk/\#/newApplication?applicationName=scorekeep&solutionStackName=Java](https://console.aws.amazon.com/elasticbeanstalk/#/newApplication?applicationName=scorekeep&solutionStackName=Java)

1. Choose **Create application** to create an application with an environment running the Java 8 SE platform\.

1. When your environment is ready, the console redirects you to the environment Dashboard\.

1. Click the URL at the top of the page to open the site\.

The instances in your environment need permission to send data to the AWS X\-Ray service\. Additionally, the sample application uses Amazon S3 and DynamoDB\. Modify the default Elastic Beanstalk instance profile to include permissions to use these services\.

**To add X\-Ray, Amazon S3 and DynamoDB permissions to your Elastic Beanstalk environment**

1. Open the Elastic Beanstalk instance profile in the IAM console: [aws\-elasticbeanstalk\-ec2\-role](https://console.aws.amazon.com/iam/home#roles/aws-elasticbeanstalk-ec2-role)\.

1. Choose **Attach Policies**\.

1. Attach **AWSXrayFullAccess**, **AmazonS3FullAccess**, and **AmazonDynamoDBFullAccess** to the role\.
**Note**  
Full access policies are not best practice for general use\. For instructions on configuring a policy with least privilege to reduce security risks, see [Optional: Least privilege policy](#xray-gettingstarted-security)\.

## Deploy to Elastic Beanstalk and generate trace data<a name="xray-gettingstarted-deploy"></a>

Deploy the sample application to your Elastic Beanstalk environment\. The sample application uses Elastic Beanstalk configuration files to configure the environment for use with X\-Ray and create the DynamoDB that it uses automatically\.

**To deploy the source code**

1. Download the sample app: [eb\-java\-scorekeep\-xray\-gettingstarted\-v1\.3\.zip](https://github.com/awslabs/eb-java-scorekeep/releases/download/xray-gs-v1.3/eb-java-scorekeep-xray-gettingstarted-v1.3.zip)

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-console.html) for your environment\.

1. Choose **Upload and Deploy**\.

1. Upload eb\-java\-scorekeep\-xray\-gettingstarted\-v1\.3\.zip, and then choose **Deploy**\.

The sample application includes a front\-end web app\. Use the web app to generate traffic to the API and send trace data to X\-Ray\.

**To generate trace data**

1. In the environment Dashboard, click the URL to open the web app\.

1. Choose **Create** to create a user and session\.

1. Type a **game name**, set the **Rules** to **Tic Tac Toe**, and then choose **Create** to create a game\.

1. Choose **Play** to start the game\.

1. Choose a tile to make a move and change the game state\.

Each of these steps generates HTTP requests to the API, and downstream calls to DynamoDB to read and write user, session, game, move, and state data\.

## View the service map in the X\-Ray console<a name="xray-gettingstarted-console"></a>

You can see the service map and traces generated by the sample application in the X\-Ray console\.

**To use the X\-Ray console**

1. Open the service map page of the [X\-Ray console](https://console.aws.amazon.com/xray/home#/service-map?timeRange=PT1H)\.

1. The console shows a representation of the service graph that X\-Ray generates from the trace data sent by the application\.  
![\[Representation of service graph X-Ray generates from trace data sent by the application\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-before.png)

The service map shows the web app client, the API running in Elastic Beanstalk, the DynamoDB service, and each DynamoDB table that the application uses\. Every request to the application, up to a configurable maximum number of requests per second, is traced as it hits the API, generates requests to downstream services, and completes\.

You can choose any node in the service graph to view traces for requests that generated traffic to that node\. Currently, the Amazon SNS node is red\. Drill down to find out why\.

**To find the cause of the error**

1. Choose the node named **SNS**\. The **Service details panel** opens to the right\.

1. Choose **View traces** to access the **Trace overview** screen\.

1. Choose the trace from the **Trace list**\. This trace doesn't have a method or URL because it was recorded during startup instead of in response to an incoming request\.  
![\[Choosing a trace from the trace list\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-tracelist-sns.png)

1. Choose the red status icon to open the **Exceptions** page for the SNS subsegment\.  
![\[Choosing the red status icon opens the Exceptions page for the SNS subsegment\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-timeline-sns.png)

1. The X\-Ray SDK automatically captures exceptions thrown by instrumented AWS SDK clients and records the stack trace\.  
![\[Exceptions tab showing captured exceptions and recorded stack trace\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-exception.png)

The cause indicates that the email address provided in a call to `createSubscription` made in the `WebConfig` class was invalid\. Let's fix that\.

## Configuring Amazon SNS notifications<a name="xray-gettingstarted-notifications"></a>

Scorekeep uses Amazon SNS to send notifications when users complete a game\. When the application starts up, it tries to create a subscription for an email address defined in an environment variable\. That call is currently failing, causing the Amazon SNS node in your service map to be red\. Configure a notification email in an environment variable to enable notifications and make the service map green\.

**To configure Amazon SNS notifications for scorekeep**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-console.html) for your environment\.

1. Choose **Configuration**\.

1. Choose **Software Configuration**\.

1. Under **Environment Properties**, replace the default value with your email address\.  
![\[Setting environment properties\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-beanstalk-envvars.png)
**Note**  
The default value uses an AWS CloudFormation [function](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions-functions.html) to retrieve a parameter stored in a configuration file \(a dummy value in this case\)\.

1. Choose **Apply**\.

When the update completes, Scorekeep restarts and creates a subscription to the SNS topic\. Check your email and confirm the subscription to see updates when you complete a game\.

![\[Service map after updates\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-after.png)

## Explore the sample application<a name="xray-gettingstarted-sample"></a>

The sample application is an HTTP web API in Java that is configured to use the X\-Ray SDK for Java\. When you deploy the application to Elastic Beanstalk, it creates the DynamoDB tables, compiles the API with Gradle, and configures the nginx proxy server to serve the web app statically at the root path\. At the same time, Elastic Beanstalk routes requests to paths starting with `/api` to the API\.

To instrument incoming HTTP requests, the application adds the `TracingFilter` provided by the SDK\.

**Example src/main/java/scorekeep/WebConfig\.java \- servlet filter**  

```
import javax.servlet.Filter;
import [com\.amazonaws\.xray\.javax\.servlet\.AWSXRayServletFilter](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html);
...

@Configuration
public class WebConfig {

  @Bean
  public Filter TracingFilter() {
    return new AWSXRayServletFilter("Scorekeep");
  }
...
```

This filter sends trace data about all incoming requests that the application serves, including request URL, method, response status, start time, and end time\.

![\[Primary service node in console\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-rootnode-gettingstarted.png)

The application also makes downstream calls to DynamoDB using the AWS SDK for Java\. To instrument these calls, the application simply takes the AWS SDK\-related submodules as dependencies, and the X\-Ray SDK for Java automatically instruments all AWS SDK clients\.

The application uses a `Buildfile` file to build the source code on\-instance with `Gradle` and a `Procfile` file to run the executable JAR that Gradle generates\. `Buildfile` and `Procfile` support is a feature of the [Elastic Beanstalk Java SE platform](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-platform.html)\.

**Example Buildfile**  

```
build: gradle build
```

**Example Procfile**  

```
web: java -Dserver.port=5000 -jar build/libs/scorekeep-api-1.0.0.jar
```

The `build.gradle` file downloads the SDK submodules from Maven during compilation by declaring them as dependencies\.

**Example build\.gradle \-\- dependencies**  

```
...
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile('org.springframework.boot:spring-boot-starter-test')
    compile('com.amazonaws:aws-java-sdk-dynamodb')
    compile("com.amazonaws:aws-xray-recorder-sdk-core")
    compile("com.amazonaws:aws-xray-recorder-sdk-aws-sdk")
    compile("com.amazonaws:aws-xray-recorder-sdk-aws-sdk-instrumentor")
    ...
}
dependencyManagement {
    imports {
        mavenBom("com.amazonaws:aws-java-sdk-bom:1.11.67")
        mavenBom("com.amazonaws:aws-xray-recorder-sdk-bom:2.4.0")
    }
}
```

The core, AWS SDK, and AWS SDK Instrumentor submodules are all that's required to automatically instrument any downstream calls made with the AWS SDK\.

To run the X\-Ray daemon, the application uses another feature of Elastic Beanstalk, configuration files\. The configuration file tells Elastic Beanstalk to run the daemon and send its log on demand\.

**Example \.ebextensions/xray\.config**  

```
option_settings:
  aws:elasticbeanstalk:xray:
    XRayEnabled: true

files:
  "/opt/elasticbeanstalk/tasks/taillogs.d/xray-daemon.conf" :
    mode: "000644"
    owner: root
    group: root
    content: |
      /var/log/xray/xray.log
```

The X\-Ray SDK for Java provides a class named `AWSXRay` that provides the global recorder, a `TracingHandler` that you can use to instrument your code\. You can configure the global recorder to customize the `AWSXRayServletFilter` that creates segments for incoming HTTP calls\. The sample includes a static block in the `WebConfig` class that configures the global recorder with plugins and sampling rules\.

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

This example uses the builder to load sampling rules from a file named `sampling-rules.json`\. Sampling rules determine the rate at which the SDK records segments for incoming requests\. 

**Example src/main/java/resources/sampling\-rules\.json**  

```
{
  "version": 1,
  "rules": [
    {
      "description": "Resource creation.",
      "service_name": "*",
      "http_method": "POST",
      "url_path": "/api/*",
      "fixed_target": 1,
      "rate": 1.0
    },
    {
      "description": "Session polling.",
      "service_name": "*",
      "http_method": "GET",
      "url_path": "/api/session/*",
      "fixed_target": 0,
      "rate": 0.05
    },
    {
      "description": "Game polling.",
      "service_name": "*",
      "http_method": "GET",
      "url_path": "/api/game/*/*",
      "fixed_target": 0,
      "rate": 0.05
    },
    {
      "description": "State polling.",
      "service_name": "*",
      "http_method": "GET",
      "url_path": "/api/state/*/*/*",
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

The sampling rules file defines four custom sampling rules and the default rule\. For each incoming request, the SDK evaluates the custom rules in the order in which they are defined\. The SDK applies the first rule that matches the request's method, path, and service name\. For Scorekeep, the first rule catches all POST requests \(resource creation calls\) by applying a fixed target of one request per second and a rate of 1\.0, or 100 percent of requests after the fixed target is satisfied\.

The other three custom rules apply a five percent rate with no fixed target to session, game, and state reads \(GET requests\)\. This minimizes the number of traces for periodic calls that the front end makes automatically every few seconds to ensure the content is up to date\. For all other requests, the file defines a default rate of one request per second and a rate of 10 percent\.

The sample application also shows how to use advanced features such as manual SDK client instrumentation, creating additional subsegments, and outgoing HTTP calls\. For more information, see [AWS X\-Ray sample application](xray-scorekeep.md)\.

## Optional: Least privilege policy<a name="xray-gettingstarted-security"></a>

 You have just deployed this tutorial using the **AmazonS3FullAccess** and **AmazonDynamoDBFullAccess** security policies\. Using a full access policy isn't the best practice in the long term\. To improve the security of what you deployed, follow these steps to update your permissions\. To learn more about security best practices in IAM policies, see [Identity and access management for AWS X\-Ray](https://docs.aws.amazon.com/xray/latest/devguide/security-iam.html)\.

To update your policies, first you identify the ARNs of your Amazon S3 and DynamoDB resources\. Then you use the ARNs in two custom IAM policies\. Finally, you apply those policies to your instance profile\.

**To identify your Amazon S3 resource**

1. Open the [Resources page of the AWS Config console](https://console.aws.amazon.com/config/home#/resources)\.

1. Under **Resource type**, filter by **AWS S3 Bucket** to find the ARN of the Amazon S3 bucket that your application uses\.

1. Choose the **Resource identifier** that's attached to `elasticbeanstalk`\.

1. Record its full **Amazon resource name**\.

1. Insert the ARN into the following IAM policy\.  
**Example**  

   ```
   {
      "Version": "2012-10-17",
      "Statement": [
      {
         "Sid": "ScorekeepS3",
         "Action": [
            "s3:GetObject",
            "s3:PutObject"
         ],
         "Effect": "Allow",
         "Resource": "arn:aws:s3:::elasticbeanstalk-region-0987654321"
      }
      ]
   }
   ```

**To identify your DynamoDB resource**

1. Open the [Resources page of the AWS Config console](https://console.aws.amazon.com/config/home#/resources)\.

1. Under **Resource type**, filter by **AWS DynamoDB Table** to find the ARN of the DynamoDB tables that your application uses\.

1. Choose the **Resource identifier** that's attached to one of the `scorekeep` tables\.

1. Record its full **Amazon resource name**\.

1. Insert the ARN into the following IAM policy\.  
**Example**  

   ```
   {
      "Version": "2012-10-17",
      "Statement": [
      {
         "Sid": "ScorekeepDynamoDB",
         "Effect": "Allow",
         "Action": [
            "dynamodb:PutItem",
            "dynamodb:UpdateItem",
            "dynamodb:DeleteItem",
            "dynamodb:GetItem"
         ],
         "Resource": "arn:aws:dynamodb:region:1234567890:table/scorekeep-*"
      }
      ]
   }
   ```

   The tables that the application creates follow a consistent naming convention\. You can use the format of `scorekeep-*` to indicate all tables following that convention\.

**To change your IAM policy**

1. Open the Elastic Beanstalk instance profile in the IAM console: [aws\-elasticbeanstalk\-ec2\-role](https://console.aws.amazon.com/iam/home#roles/aws-elasticbeanstalk-ec2-role)\.

1. Remove the **AmazonS3FullAccess** and **AmazonDynamoDBFullAccess** policies from the role\.

1. Choose **Attach policies**, and then **Create policy**\.

1. Choose the **JSON** and paste in one of the policies created previously\.

1. Choose **Review policy**\.

1. For **Name**, assign a name\.

1. Choose **Create policy**\. 

1. Assign the newly created policy to the `aws-elasticbeanstalk-ec2-role`\.

1. Repeat for the second policy created previously\.

## Clean up<a name="xray-gettingstarted-cleanup"></a>

Terminate your Elastic Beanstalk environment to shut down the Amazon EC2 instances, DynamoDB tables, and other resources\.

**To terminate your Elastic Beanstalk environment**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-console.html) for your environment\.

1. Choose **Actions**\.

1. Choose **Terminate Environment**\.

1. Choose **Terminate**\.

Trace data is automatically deleted from X\-Ray after 30 days\.

## Next steps<a name="xray-gettingstarted-nextsteps"></a>

Learn more about X\-Ray in the next chapter, [AWS X\-Ray concepts](xray-concepts.md)\.

To instrument your own app, learn more about the X\-Ray SDK for Java or one of the other X\-Ray SDKs:
+ **X\-Ray SDK for Java** – [AWS X\-Ray SDK for Java](xray-sdk-java.md)
+ **X\-Ray SDK for Node\.js** – [AWS X\-Ray SDK for Node\.js](xray-sdk-nodejs.md)
+ **X\-Ray SDK for \.NET** – [AWS X\-Ray SDK for \.NET](xray-sdk-dotnet.md)

To run the X\-Ray daemon locally or on AWS, see [AWS X\-Ray daemon](xray-daemon.md)\.

To contribute to the sample application on GitHub, see [eb\-java\-scorekeep](https://github.com/awslabs/eb-java-scorekeep/tree/xray)\.