# Getting started with the Scorekeep sample application<a name="scorekeep-tutorial"></a>

This tutorial uses the `xray-gettingstarted` branch of the [Scorekeep sample application](xray-scorekeep.md), which uses AWS CloudFormation to create and configure the resources that run the sample application and X\-Ray daemon on Amazon ECS\. The application uses the Spring framework to implement a JSON web API and the AWS SDK for Java to persist data to Amazon DynamoDB\. A servlet filter in the application instruments all incoming requests served by the application, and a request handler on the AWS SDK client instruments downstream calls to DynamoDB\.

You can follow this tutorial using either the AWS console or the AWS CLI\.

**Topics**
+ [Prerequisites](#xray-gettingstarted-prereqs)
+ [Install the Scorekeep application using CloudFormation](#xray-gettingstarted-deploy)
+ [Generate trace data](#xray-gettingstarted-generate-traces)
+ [View the service map in the AWS console](#xray-gettingstarted-console)
+ [Configuring Amazon SNS notifications](#xray-gettingstarted-notifications)
+ [Explore the sample application](#xray-gettingstarted-sample)
+ [Optional: Least privilege policy](#xray-gettingstarted-security)
+ [Clean up](#xray-gettingstarted-cleanup)
+ [Next steps](#xray-gettingstarted-nextsteps)

## Prerequisites<a name="xray-gettingstarted-prereqs"></a>

This tutorial uses AWS CloudFormation to create and configure the resources that run the sample application and X\-Ray daemon\. The following prerequisites are required to install and run through the tutorial: 

1. If you use an IAM user with limited permissions, add the following user policies in the [IAM console](https://console.aws.amazon.com/iam): 
   + `AWSCloudFormationFullAccess` – to access and use CloudFormation
   + `AmazonS3FullAccess` – to upload a template file to CloudFormation using the AWS console
   + `IAMFullAccess` – to create the Amazon ECS and Amazon EC2 instance roles
   + `AmazonEC2FullAccess` – to create the Amazon EC2 resources
   + `AmazonDynamoDBFullAccess` – to create the DynamoDB tables
   + `AmazonECS_FullAccess` – to create Amazon ECS resources
   + `AmazonSNSFullAccess` – to create the Amazon SNS topic
   + `AWSXrayReadOnlyAccess` – for permission to view the service map and traces in the X\-Ray console

1. To run through the tutorial using the AWS CLI, [install the CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) version 2\.7\.9 or later, and [configure the CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) with the user from the previous step\. Make sure the region is configured when configuring the AWS CLI with the user\. If a region is not configured, you will need to append `--region AWS-REGION` to every CLI command\. 

1. Ensure that [Git](https://github.com/git-guides/install-git) is installed, in order to clone the sample application repo\. 

1. Clone the `xray-gettingstarted` branch of the Scorekeep repo: 

   ```
   git clone https://github.com/aws-samples/eb-java-scorekeep.git xray-scorekeep -b xray-gettingstarted
   ```

## Install the Scorekeep application using CloudFormation<a name="xray-gettingstarted-deploy"></a>

------
#### [ AWS console ]

**Install the sample application using the AWS console**

1. Open the [CloudFormation console](https://console.aws.amazon.com/cloudformation/)

1. Choose **Create stack** and then choose **With new resources** from the drop\-down menu\.

1. In the **Specify template** section, choose **Upload a template file**\.

1. Select **Choose file**, navigate to the `xray-scorekeep/cloudformation` folder that was created when you cloned the git repo, and choose the `cf-resources.yaml` file\.

1. Choose **Next** to continue\.

1. Enter `scorekeep` into the **Stack name** textbox, and then choose **Next** at the bottom of the page to continue\. Note that the rest of this tutorial assumes the stack is named `scorekeep`\.

1. Scroll to the bottom of the **Configure stack options** page and choose **Next** to continue\.

1. Scroll to the bottom of the **Review** page, choose the check\-box acknowledging that CloudFormation may create IAM resources with custom names, and choose **Create stack**\.

1. The CloudFormation stack is now being created\. The stack status will be `CREATE_IN_PROGRESS` for about five minutes before changing to `CREATE_COMPLETE`\. The status will refresh periodically, or you can refresh the page\.

------
#### [ AWS CLI ]

**Install the sample application using the AWS CLI**

1. Navigate to the `cloudformation` folder of the `xray-scorekeep` repository that you cloned earlier in the tutorial:

   ```
   cd xray-scorekeep/cloudformation/
   ```

1. Enter the following AWS CLI command to create the CloudFormation stack:

   ```
   aws cloudformation create-stack --stack-name scorekeep --capabilities "CAPABILITY_NAMED_IAM" --template-body file://cf-resources.yaml
   ```

1. Wait until the CloudFormation stack status is `CREATE_COMPLETE`, which will take about five minutes\. Use the following AWS CLI command to check on the status:

   ```
   aws cloudformation describe-stacks --stack-name scorekeep --query "Stacks[0].StackStatus"
   ```

------

## Generate trace data<a name="xray-gettingstarted-generate-traces"></a>

The sample application includes a front\-end web app\. Use the web app to generate traffic to the API and send trace data to X\-Ray\. First, retrieve the web app URL using the AWS console or the AWS CLI:

------
#### [ AWS console ]

**Find the application URL using the AWS console**

1. Open the [CloudFormation console](https://console.aws.amazon.com/cloudformation/)

1. Choose the `scorekeep` stack from the list\.

1. Choose the **Outputs** tab on the `scorekeep` stack page, and choose the `LoadBalancerUrl` URL link to open the web application\.

------
#### [ AWS CLI ]

**Find the application URL using the AWS CLI**

1. Use the following command to display the URL of the web application:

   ```
   aws cloudformation describe-stacks --stack-name scorekeep --query "Stacks[0].Outputs[0].OutputValue"
   ```

1. Copy this URL and open in a browser to display the Scorekeep web application\.

------

**Use the web application to generate trace data**

1. Choose **Create** to create a user and session\.

1. Type a **game name**, set the **Rules** to **Tic Tac Toe**, and then choose **Create** to create a game\.

1. Choose **Play** to start the game\.

1. Choose a tile to make a move and change the game state\.

Each of these steps generates HTTP requests to the API, and downstream calls to DynamoDB to read and write user, session, game, move, and state data\.

## View the service map in the AWS console<a name="xray-gettingstarted-console"></a>

You can see the service map and traces generated by the sample application in the X\-Ray and CloudWatch consoles\.

------
#### [ X\-Ray console ]

**Use the X\-Ray console**

1. Open the service map page of the [X\-Ray console](https://console.aws.amazon.com/xray/home#/service-map)\.

1. The console shows a representation of the service graph that X\-Ray generates from the trace data sent by the application\. Be sure to adjust the time period of the service map if needed, to make sure that it will display all traces since you first started the web application\.  
![\[X-Ray service map time period\]](http://docs.aws.amazon.com/xray/latest/devguide/images/xray-console-time-period-15-minutes.png)

The service map shows the web app client, the API running in Amazon ECS, and each DynamoDB table that the application uses\. Every request to the application, up to a configurable maximum number of requests per second, is traced as it hits the API, generates requests to downstream services, and completes\.

You can choose any node in the service graph to view traces for requests that generated traffic to that node\. Currently, the Amazon SNS node is yellow\. Drill down to find out why\.

![\[X-Ray console service map page\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-servicemap-before-ECS.png)

**To find the cause of the error**

1. Choose the node named **SNS**\. The node details panel is displayed\.

1. Choose **View traces** to access the **Trace overview** screen\.

1. Choose the trace from the **Trace list**\. This trace doesn't have a method or URL because it was recorded during startup instead of in response to an incoming request\.  
![\[Choosing a trace from the trace list\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-tracelist-sns.png)

1. Choose the error status icon within the Amazon SNS segment at the bottom of the page, to open the **Exceptions** page for the SNS subsegment\.  
![\[Choose the error status icon to open the Exceptions page for the Amazon SNS subsegment\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-timeline-sns-ecs.png)

1. The X\-Ray SDK automatically captures exceptions thrown by instrumented AWS SDK clients and records the stack trace\.  
![\[Exceptions tab showing captured exceptions and recorded stack trace\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-exception.png)

------
#### [ CloudWatch console ]

**Use the CloudWatch console**

1. Open the [X\-Ray service map](https://console.aws.amazon.com/cloudwatch/home#xray:service-map/map) page of the CloudWatch console\.

1. The console shows a representation of the service graph that X\-Ray generates from the trace data sent by the application\. Be sure to adjust the time period of the service map if needed, to make sure that it will display all traces since you first started the web application\.  
![\[CloudWatch service map time period\]](http://docs.aws.amazon.com/xray/latest/devguide/images/cw-console-service-map-time-period-15-minutes.png)

The service map shows the web app client, the API running in Amazon EC2, and each DynamoDB table that the application uses\. Every request to the application, up to a configurable maximum number of requests per second, is traced as it hits the API, generates requests to downstream services, and completes\.

You can choose any node in the service graph to view traces for requests that generated traffic to that node\. Currently, the Amazon SNS node is orange\. Drill down to find out why\.

![\[X-Ray console service map page\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-cw-servicemap-before-ECS.png)

**To find the cause of the error**

1. Choose the node named **SNS**\. The SNS node details panel is displayed below the map\.

1. Choose **View traces** to access the **Traces** page\.

1. Add the bottom of the page, choose the trace from the **Traces** list\. This trace doesn't have a method or URL because it was recorded during startup instead of in response to an incoming request\.  
![\[Choosing a trace from the trace list\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-cw-tracelist-sns-ecs.png)

1. Choose the Amazon SNS subsegment at the bottom of the segments timeline, and choose the **Exceptions** tab for the SNS subsegment to view the exception details\.  
![\[View the Exceptions tab for the Amazon SNS subsegment\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-gettingstarted-cw-timeline-sns-ecs.png)

------

The cause indicates that the email address provided in a call to `createSubscription` made in the `WebConfig` class was invalid\. In the next section, we'll fix that\.

## Configuring Amazon SNS notifications<a name="xray-gettingstarted-notifications"></a>

Scorekeep uses Amazon SNS to send notifications when users complete a game\. When the application starts up, it tries to create a subscription for an email address defined in a CloudFormation stack parameter\. That call is currently failing\. Configure a notification email to enable notifications, and resolve the failures highlighted in the service map\.

------
#### [ AWS console ]

**To configure Amazon SNS notifications using the AWS console**

1. Open the [CloudFormation console](https://console.aws.amazon.com/cloudformation/)

1. Choose the radio button next to the `scorekeep` stack name in the list, and then choose **Update**\.

1. Make sure that **Use current template** is chosen, and then click **Next** on the **Update stack** page\.

1. Find the **Email** parameter in the list, and replace the default value with a valid email address\.  
![\[Update email configuration\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-cf-email-update.png)

1. Scroll to the bottom of the page and choose **Next**\.

1. Scroll to the bottom of the **Review** page, choose the check\-box acknowledging that CloudFormation may create IAM resources with custom names, and choose **Update stack**\.

1. The CloudFormation stack is now being updated\. The stack status will be `UPDATE_IN_PROGRESS` for about five minutes before changing to `UPDATE_COMPLETE`\. The status will refresh periodically, or you can refresh the page\.

------
#### [ AWS CLI ]

**To configure Amazon SNS notifications using the AWS CLI**

1. Navigate to the `xray-scorekeep/cloudformation/` folder you previously created, and open the `cf-resources.yaml` file in a text editor\.

1. Find the `Default` value within the **Email** parameter and change it from *UPDATE\_ME* to a valid email address\.

   ```
   Parameters:
     Email:
       Type: String
       Default: UPDATE_ME # <- change to a valid abc@def.xyz email address
   ```

1. From the `cloudformation` folder, update the CloudFormation stack with the following AWS CLI command: 

   ```
   aws cloudformation update-stack --stack-name scorekeep --capabilities "CAPABILITY_NAMED_IAM" --template-body file://cf-resources.yaml
   ```

1. Wait until the CloudFormation stack status is `UPDATE_COMPLETE`, which will take a few minutes\. Use the following AWS CLI command to check on the status:

   ```
   aws cloudformation describe-stacks --stack-name scorekeep --query "Stacks[0].StackStatus"
   ```

------

When the update completes, Scorekeep restarts and creates a subscription to the SNS topic\. Check your email and confirm the subscription to see updates when you complete a game\. Open the service map to verify that the calls to SNS are no longer failing\.

## Explore the sample application<a name="xray-gettingstarted-sample"></a>

The sample application is an HTTP web API in Java that is configured to use the X\-Ray SDK for Java\. When you deploy the application with the CloudFormation template, it creates the DynamoDB tables, Amazon ECS Cluster, and other services required to run Scorekeep on ECS\. A task definition file for ECS is created through CloudFormation\. This file defines the container images used per task in an ECS cluster\. These images are obtained from the official X\-Ray public ECR\. The scorekeep API container image has the API compiled with Gradle\. The container image of the Scorekeep frontend container serves the frontend using the nginx proxy server\. This server routes requests to paths starting with /api to the API\.

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

The application also makes downstream calls to DynamoDB using the AWS SDK for Java\. To instrument these calls, the application simply takes the AWS SDK\-related submodules as dependencies, and the X\-Ray SDK for Java automatically instruments all AWS SDK clients\.

The application uses `Docker` to build the source code on\-instance with the `Gradle Docker Image` and the `Scorekeep API Dockerfile` file to run the executable JAR that Gradle generates at its `ENTRYPOINT`\. 

**Example use of Docker to build via Gradle Docker Image**  

```
docker run --rm -v /PATH/TO/SCOREKEEP_REPO/home/gradle/project -w /home/gradle/project gradle:4.3 gradle build
```

**Example Dockerfile ENTRYPOINT**  

```
ENTRYPOINT [ "sh", "-c", "java -Dserver.port=5000 -jar scorekeep-api-1.0.0.jar" ]
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
        mavenBom("com.amazonaws:aws-xray-recorder-sdk-bom:2.11.0")
    }
}
```

The core, AWS SDK, and AWS SDK Instrumentor submodules are all that's required to automatically instrument any downstream calls made with the AWS SDK\.

To relay the raw segment data to the X\-Ray API, the X\-Ray daemon is required to listen for traffic on UDP port 2000\. To do so, the application has the X\-Ray daemon run in a container that is deployed alongside the Scorekeep application on ECS as a *sidecar container*\. Check out the [X\-Ray daemon](xray-daemon.md) topic for more information\.

**Example X\-Ray Daemon Container Definition in an ECS Task Definition**  

```
...
Resources:
  ScorekeepTaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties: 
      ContainerDefinitions: 
      ...
      
      - Cpu: '256'
        Essential: true
        Image: amazon/aws-xray-daemon
        MemoryReservation: '128'
        Name: xray-daemon
        PortMappings: 
          - ContainerPort: '2000'
            HostPort: '2000'
            Protocol: udp
      ...
```

The X\-Ray SDK for Java provides a class named `AWSXRay` that provides the global recorder, a `TracingHandler` that you can use to instrument your code\. You can configure the global recorder to customize the `AWSXRayServletFilter` that creates segments for incoming HTTP calls\. The sample includes a static block in the `WebConfig` class that configures the global recorder with plugins and sampling rules\.

**Example src/main/java/scorekeep/WebConfig\.java \- recorder**  

```
import com.amazonaws.xray.AWSXRay;
import com.amazonaws.xray.AWSXRayRecorderBuilder;
import com.amazonaws.xray.javax.servlet.AWSXRayServletFilter;
import com.amazonaws.xray.plugins.ECSPlugin;
import com.amazonaws.xray.plugins.EC2Plugin;
import com.amazonaws.xray.strategy.sampling.LocalizedSamplingStrategy;
...

@Configuration
public class WebConfig {
  ...
  
  static {
    AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder.standard().withPlugin(new ECSPlugin()).withPlugin(new EC2Plugin());

    URL ruleFile = WebConfig.class.getResource("/sampling-rules.json");
    builder.withSamplingStrategy(new LocalizedSamplingStrategy(ruleFile));

    AWSXRay.setGlobalRecorder(builder.build());
    ...
    
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

 The Scorekeep ECS containers access resources using full access policies, such as `AmazonSNSFullAccess` and `AmazonDynamoDBFullAccess`\. Using full access policies is not the best practice for production applications\. The following example updates the DynamoDB IAM policy to improve the security of the application\. To learn more about security best practices in IAM policies, see [Identity and access management for AWS X\-Ray](security-iam.md)\.

**Example cf\-resources\.yaml template ECSTaskRole definition**  

```
ECSTaskRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement: 
          - 
            Effect: "Allow"
            Principal: 
              Service: 
                - "ecs-tasks.amazonaws.com"
            Action: 
              - "sts:AssumeRole"
      ManagedPolicyArns:
        - "arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess"
        - "arn:aws:iam::aws:policy/AmazonSNSFullAccess"
        - "arn:aws:iam::aws:policy/AWSXrayFullAccess"
      RoleName: "scorekeepRole"
```

To update your policy, first you identify the ARN of your DynamoDB resources\. Then you use the ARN in a custom IAM policy\. Finally, you apply that policy to your instance profile\.

**To identify the ARN of your DynamoDB resource:**

1. Open the [DynamoDB console](https://console.aws.amazon.com/dynamodbv2)\.

1. Choose **Tables** from the left navigation bar\.

1. Choose any of the `scorekeep-*` to display the table detail page\.

1. Under the **Overview** tab, choose **Additional info** to expand the section and view the Amazon Resource Name \(ARN\)\. Copy this value\.

1. Insert the ARN into the following IAM policy, replacing the `AWS_REGION` and `AWS_ACCOUNT_ID` values with your specific region and account ID\. This new policy allows only the actions specified, instead of the `AmazonDynamoDBFullAccess` policy which allows any action\.  
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
                   "dynamodb:GetItem",
                   "dynamodb:Scan",
                   "dynamodb:Query"
               ],
               "Resource": "arn:aws:dynamodb:<AWS_REGION>:<AWS_ACCOUNT_ID>:table/scorekeep-*"
           }
       ]
   }
   ```

   The tables that the application creates follow a consistent naming convention\. You can use the `scorekeep-*` format to indicate all Scorekeep tables\.

**Change your IAM policy**

1. Open the [Scorekeep task role \(scorekeepRole\)](https://console.aws.amazon.com/iamv2/home#/roles/details/scorekeepRole) from the IAM console\.

1. Choose the check box next to the `AmazonDynamoDBFullAccess` policy and choose **Remove** to remove this policy\. 

1. Choose **Add permissions**, and then **Attach policies**, and finally **Create policy**\.

1. Choose the **JSON** tab and paste in the policy created above\.

1. Choose **Next: Tags** at the bottom of the page\.

1. Choose **Next: Review** at the bottom of the page\.

1. For **Name**, assign a name for the policy\.

1. Choose **Create policy** at the bottom of the page\. 

1. Attach the newly created policy to the `scorekeepRole` role\. It may take a few minutes for the attached policy to take effect\.

If you have attached the new policy to the `scorekeepRole` role, you must detach it before deleting the CloudFormation stack, since this attached policy will block the stack from being deleted\. The policy can be automatically detached by deleting the policy\.

**Remove your custom IAM policy**

1. Open the [IAM console](https://console.aws.amazon.com/iam)\.

1. Choose **Policies** from the left navigation bar\.

1. Search for the custom policy name you created earlier in this section, and choose the radio button next to the policy name to highlight it\.

1. Choose the **Actions** drop\-down and then choose **Delete**\.

1. Type the name of the custom policy and then choose **Delete** to confirm deletion \. This will automatically detach the policy from the `scorekeepRole` role\.

## Clean up<a name="xray-gettingstarted-cleanup"></a>

Follow these steps to delete the Scorekeep application resources:

**Note**  
If you created and attached custom policies using the prior section of this tutorial, you must remove the policy from the `scorekeepRole` before deleting the CloudFormation stack\.

------
#### [ AWS console ]

**Delete the sample application using the AWS console**

1. Open the [CloudFormation console](https://console.aws.amazon.com/cloudformation/)

1. Choose the radio button next to the `scorekeep` stack name in the list, and then choose **Delete**\.

1. The CloudFormation stack is now being deleted\. The stack status will be `DELETE_IN_PROGRESS` for a few minutes until all resources are deleted\. The status will refresh periodically, or you can refresh the page\.

------
#### [ AWS CLI ]

**Delete the sample application using the AWS CLI**

1. Enter the following AWS CLI command to delete the CloudFormation stack:

   ```
   aws cloudformation delete-stack --stack-name scorekeep
   ```

1. Wait until the CloudFormation stack no longer exists, which will take about five minutes\. Use the following AWS CLI command to check on the status:

   ```
   aws cloudformation describe-stacks --stack-name scorekeep --query "Stacks[0].StackStatus"
   ```

------

## Next steps<a name="xray-gettingstarted-nextsteps"></a>

Learn more about X\-Ray in the next chapter, [AWS X\-Ray concepts](xray-concepts.md)\.

To instrument your own app, learn more about the X\-Ray SDK for Java or one of the other X\-Ray SDKs:
+ **X\-Ray SDK for Java** – [AWS X\-Ray SDK for Java](xray-sdk-java.md)
+ **X\-Ray SDK for Node\.js** – [AWS X\-Ray SDK for Node\.js](xray-sdk-nodejs.md)
+ **X\-Ray SDK for \.NET** – [AWS X\-Ray SDK for \.NET](xray-sdk-dotnet.md)

To run the X\-Ray daemon locally or on AWS, see [AWS X\-Ray daemon](xray-daemon.md)\.

To contribute to the sample application on GitHub, see [eb\-java\-scorekeep](https://github.com/awslabs/eb-java-scorekeep/tree/xray-gettingstarted)\.