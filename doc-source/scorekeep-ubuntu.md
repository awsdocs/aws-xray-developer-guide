# Getting started with AWS X\-Ray through the AWS CLI<a name="scorekeep-ubuntu"></a>

This tutorial shows you how to use the AWS CLI to deploy the Scorekeep sample application with an AWS CloudFormation template, and then to generate and retrieve trace data\. You use the AWS CLI to access the X\-Ray service directly and the same APIs that the X\-Ray console uses to retrieve the service graph and raw trace data\.

To access the raw service map and trace data, you use the AWS CLI to call the X\-Ray API\. The service map and trace data are in JSON format\. You can then query the trace data to ensure that your application is sending data, or to check specific fields as part of your test automation\.

This tutorial takes approximately 30 minutes to complete\. Although we provide links to learn more about various related topics, you don't need to leave this page to complete the tutorial\. We do suggest you open and review the scripts in a text editor or in [the GitHub repository](https://github.com/aws-samples/eb-java-scorekeep/tree/xray-gettingstarted) as you go\. The scripts demonstrate use cases for the AWS CLI and how to manage the data returned by API calls\.

**Topics**
+ [Prerequisites](#scorekeep-ubuntu-prereq)
+ [Create the Amazon EC2 instance](#scorekeep-ubuntu-instance)
+ [Install the AWS CLI](#scorekeep-ubuntu-cli)
+ [Deploy Scorekeep](#scorekeep-ubuntu-deploy)
+ [Generate trace data](#scorekeep-ubuntu-testdata)
+ [Get data](#scorekeep-ubuntu-getdata)
+ [Configure Amazon SNS notifications](#scorekeep-ubuntu-sns)
+ [Clean up](#scorekeep-ubuntu-cleanup)
+ [Next steps](#scorekeep-ubuntu-nextstep)

## Prerequisites<a name="scorekeep-ubuntu-prereq"></a>

The following sections describe what you should know and what permissions you must be able to access to follow this tutorial\.

### Command line<a name="scorekeep-ubuntu-tutorial-prereqs-command"></a>

This tutorial uses an Amazon Elastic Compute Cloud \(Amazon EC2\) instance of Ubuntu Server 18\.04 LTS, which is AWS Free Tier eligible\. The Ubuntu instance provides a terminal\. To follow the procedures in this tutorial, you need a command line terminal or shell to run commands\. Commands are shown in listings preceded by a prompt symbol \($\)\.

```
$ this is a command
this is output
```

You should also be familiar with working with `vi` in the terminal\. There are many information sources on `vi` where you can learn the basic commands, such as a [Vim Cheat Sheet](https://vim.rtorr.com)\.

### User permissions<a name="scorekeep-ubuntu-tutorial-prereqs-permissions"></a>

When working on an Amazon EC2 instance, the recommended way to get credentials is to use an instance profile role\. This allows you to delegate permissions to make API requests without distributing your AWS credentials\. You will create an AWS Identity and Access Management \(IAM\) role that specifies the permissions that you want to grant to the applications that run on your Amazon EC2 instance\.

Use the following steps to prepare a role to attach to your Amazon EC2 instance on creation\.

**To create a Scorekeep role**

1. Sign in to the AWS Management Console and open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. Choose **Roles**, and then choose **Create role**\.

1. Under **Common use cases**, choose **EC2**, and then choose **Next: Permissions**\.

1. Choose **AdministratorAccess**, and then choose **Next: Tags**\.

1. Choose **Next: Review**, assign the name **scorekeep\-ubuntu**, and then choose **Create role**\.

## Create the Amazon EC2 instance<a name="scorekeep-ubuntu-instance"></a>

To demonstrate how to configure the sample application using the AWS CLI, you first deploy a clean Amazon EC2 instance through the console\. The rest of the tutorial requires this instance\.

**To create an Ubuntu Amazon EC2 instance**

1. Sign in to the AWS Management Console and open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Verify you are in the AWS Region where you want to create the instance\.

1. Choose **Launch Instance**\. Use the search feature to filter for **Ubuntu**, and then choose **Ubuntu Server 18\.04 LTS**\. It's AWS Free Tier eligible\.

1. From the top of the launch wizard, choose **3\. Configure Instance**\. Then assign the `scorekeep-ubuntu` role you created in the permission prerequisites to the **IAM role** field\.

1. From the top of the launch wizard, choose **5\. Add Tags**, and then choose **Add tag**\. Next, do the following: 
   + For **Key**, enter **Name**\.
   + For **Value**, enter **scorekeep\-ubuntu**\.

1. From the top of the launch wizard, choose **6\. Configure Security Group**\. Rename **Security group name** to **scorekeep\-ubuntu**\.

1. \(Optional\) For **Source**, choose **My IP**\. Restricting access to your instance is a security best practice\.

1. Choose **Review and Launch**, and then choose **Launch**\.

1. Choose **Create a new key pair** and name it **scorekeep\-ubuntu\-key**\.

1. Choose **Download Key Pair**, and then choose **Launch Instances**\. 

This instance takes less than five minutes to launch, and you can connect soon after\.

**To connect to an Ubuntu Amazon EC2 instance**

1. Sign in to the AWS Management Console and open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Verify you're in the AWS Region where your instance was created\.

1. Choose **Instances**\.

1. In the list of instances, select **scorekeep\-ubuntu**\.

1. For instructions on connection methods, choose **Connect**\.

## Install the AWS CLI<a name="scorekeep-ubuntu-cli"></a>

After you're connected to the instance, you need to install the AWS CLI\. You use AWS CLI commands throughout the subsequent steps to create resources, deploy the instrumented application, and pull trace data on the application you deployed\. 

To configure the AWS CLI and make calls, you need an AWS access key ID and AWS secret access key\. When working on an Amazon EC2 instance, use the role attached to your instance to get these credentials\. Do not use your personal keys\.

For detailed information and scenarios on instance profiles on Amazon EC2, see [IAM roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#ec2-instance-profile) and [Using temporary credentials with AWS resources](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html#using-temp-creds-sdk-ec2-instances)\.

**To install and configure the AWS CLI**  
Run the following commands to download the AWS CLI, and then unpack and install it\.

```
$ sudo apt update
$ sudo apt install python3-pip 
$ pip3 install awscli --upgrade --user
$ sudo apt install awscli
$ aws --version
aws-cli/1.18.63 Python/3.6.9 Linux/4.15.0-1065-aws botocore/1.16.13
```

Run the following commands to get the credentials from your Amazon EC2 instance role and assign the credentials for use\.

You must also choose a Region to enter from the [Regional endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#regional-endpoints) table\. This is required\. It determines the Region that subsequent resources are created in and where scripts point to\.

```
$ curl http://169.254.169.254/latest/meta-data/iam/security-credentials/scorekeep-ubuntu
{
  "Code" : "Success",
  "LastUpdated" : "2020-04-29T01:03:10Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAIOSFODNN7EXAMPLE",
  "SecretAccessKey" : "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "Token" : "TokenString",
  "Expiration" : "2020-04-29T07:38:23Z"
}
$ export AWS_ACCESS_KEY_ID=ASIAIOSFODNN7EXAMPLE
$ export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
$ export AWS_SESSION_TOKEN=TokenString
$ export AWS_DEFAULT_REGION=Region
```

**Note**  
If you disconnect from your Amazon EC2 instance before finishing the tutorial, you need to repeat these export commands to reestablish the credentials and the Region\.

## Deploy Scorekeep<a name="scorekeep-ubuntu-deploy"></a>

The environment and application are now ready for you to deploy Scorekeep\. This [sample Java application](xray-scorekeep.md) generates segments and sends them to X\-Ray\. 

The application uses the Spring Framework to implement a JSON web API, and the AWS SDK for Java to persist data to DynamoDB\. A servlet filter in the application instruments all incoming requests served by the application\. A request handler on the AWS SDK client instruments downstream calls to DynamoDB\.

The package contains several numbered shell scripts, which streamline the creation of your resources\. The package also includes an AWS CloudFormation template\. To learn more about AWS CloudFormation, see [What is AWS CloudFormation?](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)

**To download and deploy Scorekeep**  
Run the following commands to install and configure git, and run `git clone` to clone the Scorekeep repository to your Ubuntu server\. Scorekeep has different branches for different getting\-started projects\. For this tutorial, run `git checkout xray-gettingstarted`\.

```
$ git config --global user.name "Name"
$ git config --global user.email "email@domain.com"
$ git clone https://github.com/aws-samples/eb-java-scorekeep.git
$ cd eb-java-scorekeep
eb-java-scorekeep$ git checkout xray-gettingstarted
```

Run the project's deployment scripts to create an Amazon S3 bucket and deploy the application\. Continue reading about the scripts while the project deploys\.

```
eb-java-scorekeep$ ./1-create-bucket.sh
make_bucket: beanstalk-artifacts-8174xmplbb388b50
eb-java-scorekeep$ ./2-deploy.sh
Successfully packaged artifacts and wrote output template to file out.yml.
Execute the following command to deploy the packaged template
aws cloudformation deploy --template-file /home/ubuntu/eb-java-scorekeep/out.yml --stack-name scorekeep

Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - scorekeep
```

 The `1-create-bucket.sh` script creates a bucket with the naming convention `beanstalk-artifacts-$BUCKET_ID`, where `$BUCKET_ID` is a randomly generated ID\. 

The `2-deploy.sh` script creates an AWS CloudFormation stack that contains an AWS Elastic Beanstalk environment\. It uses the AWS CLI to upload the source code to Amazon S3 and deploy a template that defines the stack's resources\. 

**Note**  
Creation of all the stack artifacts takes about 10 minutes\. You might need to press **Enter** to prompt the final success message\.

**Example [eb\-java\-scorekeep/2\-deploy\.sh](https://github.com/aws-samples/eb-java-scorekeep/blob/master/2-deploy.sh)**  

```
#!/bin/bash
set -eo pipefail
ARTIFACT_BUCKET=$(cat bucket-name.txt)
git archive --format=zip HEAD > package.zip
aws cloudformation package --template-file template.yml --s3-bucket $ARTIFACT_BUCKET --output-template-file out.yml
aws cloudformation deploy --template-file out.yml --stack-name scorekeep --capabilities CAPABILITY_NAMED_IAM
```

The `template.yml` file creates an Elastic Beanstalk environment with the required permissions, Amazon DynamoDB tables, and other resources that the sample application uses\.

**Example [eb\-java\-scorekeep/template\.yml](https://github.com/aws-samples/eb-java-scorekeep/blob/master/template.yml)**  

```
AWSTemplateFormatVersion: 2010-09-09
Description: An AWS Elastic application that uses DynamoDB.
Parameters:
  emailAddress:
    Type: String
    Default: UPDATEME
Resources:
  application:
    Type: [AWS::ElasticBeanstalk::Application](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-beanstalk.html)
    Properties:
      ApplicationName: Scorekeep
      Description: RESTful web API in Java with Spring that provides an HTTP interface for creating and managing game sessions and users.
  version:
    Type: [AWS::ElasticBeanstalk::ApplicationVersion](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-beanstalk-version.html)
    Properties:
      ApplicationName: !Ref application
      SourceBundle: ./package.zip
  environment:
    Type: [AWS::ElasticBeanstalk::Environment](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-beanstalk-environment.html)
    Properties:
      ApplicationName: !Ref application
      EnvironmentName: BETA
      OptionSettings:
        - Namespace: aws:elasticbeanstalk:application:environment
          OptionName: AWS_REGION
          Value: !Ref AWS::Region
        ...
```
When the deployment completes, run `3-open-website.sh` to get the site URL\.  

```
eb-java-scorekeep$ ./3-open-website.sh
http://awseb-e-b-AWSEBLoa-SR79XMPLF2H8-586716793.us-west-2.elb.amazonaws.com
```
Open the website in a browser to see the web app and start generating trace data\.

## Generate trace data<a name="scorekeep-ubuntu-testdata"></a>

You can also use the `test-api.sh` script included in the pacakge to run end\-to\-end scenarios and generate diverse trace data while you test the API\.

**To use the `test-api.sh` script**

1. Install the `jq` library\. The `test-api.sh` script uses `jq` to parse JSON returned by API calls\.

   ```
   $ sudo apt install jq
   ```

1. Use the AWS CLI to get your environment's **CNAME**\. Use the `EnvironmentName` **BETA** to query\. This is the name that was defined in the AWS CloudFormation template\.

   ```
   $ aws elasticbeanstalk describe-environments --environment-names BETA
   {
       "Environments": [
           {
               "EnvironmentName": "BETA",
               "EnvironmentId": "e-fn2pvynnue",
               "ApplicationName": "Scorekeep",
               "VersionLabel": "scorekeep-version-1jd6hjta4qjzl",
               "SolutionStackName": "64bit Amazon Linux 2018.03 v2.10.4 running Java 8",
               "PlatformArn": "arn:aws:elasticbeanstalk:us-west-2::platform/Java 8 running on 64bit Amazon Linux/2.10.4",
               "EndpointURL": "awseb-e-f-AWSEBLoa-1UJJGXA6MKWMN-1234567.us-west-2.elb.amazonaws.com",
               "CNAME": "BETA.eba-example.us-west-2.elasticbeanstalk.com",
   			...
   }
   ```

1. Open `/bin/test-api.sh` and replace the value for API with your environment's URL\.

   ```
   eb-java-scorekeep$ vi bin/test-api.sh
   				
   #!/bin/bash
   API=scorekeep-ubuntu.9hbtbm23t2.us-west-2.elasticbeanstalk.com/api
   ```

1. Run the script to generate traffic to the API\.

   ```
   eb-java-scorekeep$ ./bin/test-api.sh
   Creating users,
   session,
   game,
   configuring game,
   playing game,
   ending game,
   game complete.
   {"id":"MTBP8BAS","session":"HUF6IT64","name":"tic-tac-toe-test","users":["QFF3HBGM","KL6JR98D"],"rules":"102","startTime":1476314241,"endTime":1476314245,"states":["JQVLEOM2","D67QLPIC","VF9BM9NC","OEAA6GK9","2A705O73","1U2LFTLJ","HUKIDD70","BAN1C8FI","G3UDJTUF","AB70HVEV"],"moves":["BS8F8LQ","4MTTSPKP","463OETES","SVEBCL3N","N7CQ1GHP","O84ONEPD","EG4BPROQ","V4BLIDJ3","9RL3NPMV"]}
   ```

## Get data<a name="scorekeep-ubuntu-getdata"></a>

You can use the [https://docs.aws.amazon.com/xray/latest/api/API_GetServiceGraph.html](https://docs.aws.amazon.com/xray/latest/api/API_GetServiceGraph.html) API to retrieve the JSON service graph\. The API requires a start time and end time\. You can calculate these from a Linux terminal by using the `date` command\.

```
$ date +%s
1499394617
```

`date +%s` prints a date in seconds\. Use this number as an end time, and subtract time from it to create a start time\.

**Example script to retrieve a service graph for the last 10 minutes**  

```
$ EPOCH=$(date +%s)
$ aws xray get-service-graph --start-time $(($EPOCH-600)) --end-time $EPOCH
```

The following example shows a service graph with four nodes, including a client node, an EC2 instance, a DynamoDB table, and an Amazon Simple Notification Service \(Amazon SNS\) topic\.

### Raw service graph<a name="scorekeep-ubuntu-tutorial-service-graph"></a>

**Example GetServiceGraph output**  

```
{
    "Services": [
        {
            "ReferenceId": 0,
            "Name": "xray-sample.elasticbeanstalk.com",
            "Names": [
                "xray-sample.elasticbeanstalk.com"
            ],
            "Type": "client",
            "State": "unknown",
            "StartTime": 1528317567.0,
            "EndTime": 1528317589.0,
            "Edges": [
                {
                    "ReferenceId": 2,
                    "StartTime": 1528317567.0,
                    "EndTime": 1528317589.0,
                    "SummaryStatistics": {
                        "OkCount": 3,
                        "ErrorStatistics": {
                            "ThrottleCount": 0,
                            "OtherCount": 1,
                            "TotalCount": 1
                        },
                        "FaultStatistics": {
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "TotalCount": 4,
                        "TotalResponseTime": 0.273
                    },
                    "ResponseTimeHistogram": [
                        {
                            "Value": 0.005,
                            "Count": 1
                        },
                        {
                            "Value": 0.015,
                            "Count": 1
                        },
                        {
                            "Value": 0.157,
                            "Count": 1
                        },
                        {
                            "Value": 0.096,
                            "Count": 1
                        }
                    ],
                    "Aliases": []
                }
            ]
        },
        {
            "ReferenceId": 1,
            "Name": "awseb-e-dixzws4s9p-stack-StartupSignupsTable-4IMSMHAYX2BA",
            "Names": [
                "awseb-e-dixzws4s9p-stack-StartupSignupsTable-4IMSMHAYX2BA"
            ],
            "Type": "AWS::DynamoDB::Table",
            "State": "unknown",
            "StartTime": 1528317583.0,
            "EndTime": 1528317589.0,
            "Edges": [],
            "SummaryStatistics": {
                "OkCount": 2,
                "ErrorStatistics": {
                    "ThrottleCount": 0,
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "FaultStatistics": {
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "TotalCount": 2,
                "TotalResponseTime": 0.12
            },
            "DurationHistogram": [
                {
                    "Value": 0.076,
                    "Count": 1
                },
                {
                    "Value": 0.044,
                    "Count": 1
                }
            ],
            "ResponseTimeHistogram": [
                {
                    "Value": 0.076,
                    "Count": 1
                },
                {
                    "Value": 0.044,
                    "Count": 1
                }
            ]
        },
        {
            "ReferenceId": 2,
            "Name": "xray-sample.elasticbeanstalk.com",
            "Names": [
                "xray-sample.elasticbeanstalk.com"
            ],
            "Root": true,
            "Type": "AWS::EC2::Instance",
            "State": "active",
            "StartTime": 1528317567.0,
            "EndTime": 1528317589.0,
            "Edges": [
                {
                    "ReferenceId": 1,
                    "StartTime": 1528317567.0,
                    "EndTime": 1528317589.0,
                    "SummaryStatistics": {
                        "OkCount": 2,
                        "ErrorStatistics": {
                            "ThrottleCount": 0,
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "FaultStatistics": {
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "TotalCount": 2,
                        "TotalResponseTime": 0.12
                    },
                    "ResponseTimeHistogram": [
                        {
                            "Value": 0.076,
                            "Count": 1
                        },
                        {
                            "Value": 0.044,
                            "Count": 1
                        }
                    ],
                    "Aliases": []
                },
                {
                    "ReferenceId": 3,
                    "StartTime": 1528317567.0,
                    "EndTime": 1528317589.0,
                    "SummaryStatistics": {
                        "OkCount": 2,
                        "ErrorStatistics": {
                            "ThrottleCount": 0,
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "FaultStatistics": {
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "TotalCount": 2,
                        "TotalResponseTime": 0.125
                    },
                    "ResponseTimeHistogram": [
                        {
                            "Value": 0.049,
                            "Count": 1
                        },
                        {
                            "Value": 0.076,
                            "Count": 1
                        }
                    ],
                    "Aliases": []
                }
            ],
            "SummaryStatistics": {
                "OkCount": 3,
                "ErrorStatistics": {
                    "ThrottleCount": 0,
                    "OtherCount": 1,
                    "TotalCount": 1
                },
                "FaultStatistics": {
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "TotalCount": 4,
                "TotalResponseTime": 0.273
            },
            "DurationHistogram": [
                {
                    "Value": 0.005,
                    "Count": 1
                },
                {
                    "Value": 0.015,
                    "Count": 1
                },
                {
                    "Value": 0.157,
                    "Count": 1
                },
                {
                    "Value": 0.096,
                    "Count": 1
                }
            ],
            "ResponseTimeHistogram": [
                {
                    "Value": 0.005,
                    "Count": 1
                },
                {
                    "Value": 0.015,
                    "Count": 1
                },
                {
                    "Value": 0.157,
                    "Count": 1
                },
                {
                    "Value": 0.096,
                    "Count": 1
                }
            ]
        },
        {
            "ReferenceId": 3,
            "Name": "SNS",
            "Names": [
                "SNS"
            ],
            "Type": "AWS::SNS",
            "State": "unknown",
            "StartTime": 1528317583.0,
            "EndTime": 1528317589.0,
            "Edges": [],
            "SummaryStatistics": {
                "OkCount": 2,
                "ErrorStatistics": {
                    "ThrottleCount": 0,
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "FaultStatistics": {
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "TotalCount": 2,
                "TotalResponseTime": 0.125
            },
            "DurationHistogram": [
                {
                    "Value": 0.049,
                    "Count": 1
                },
                {
                    "Value": 0.076,
                    "Count": 1
                }
            ],
            "ResponseTimeHistogram": [
                {
                    "Value": 0.049,
                    "Count": 1
                },
                {
                    "Value": 0.076,
                    "Count": 1
                }
            ]
        }
    ]
}
```

## Configure Amazon SNS notifications<a name="scorekeep-ubuntu-sns"></a>

Scorekeep uses Amazon SNS to send notifications when users complete a game\. When the application starts up, it tries to create a subscription for an email address defined in an environment variable\. That call is currently failing, causing the `ErrorStatistics` count in your raw data\. 

For more information about how this appears in the console, see [View the service map in the X\-Ray console](https://docs.aws.amazon.com/xray/latest/devguide/xray-gettingstarted.html#xray-gettingstarted-console)\.

The following command sets the value of the `NOTIFICATION_EMAIL` variable in the BETA environment to *email@domain\.com*\.

```
$ aws elasticbeanstalk update-environment --environment-name BETA --option-settings Namespace=aws:elasticbeanstalk:application:environment,OptionName=NOTIFICATION_EMAIL,Value=email@domain.com
{
    "EnvironmentName": "BETA",
    "EnvironmentId": "e-iarzmpigxz",
    "ApplicationName": "Scorekeep",
    "VersionLabel": "scorekeep-version-1rrbj5e9c31yc",
    "SolutionStackName": "64bit Amazon Linux 2018.03 v2.10.7 running Java 8",
    ...
}
```

When the update completes, Scorekeep restarts and creates a subscription to the Amazon SNS topic\. Check your email and confirm the subscription to see updates when you complete a game\.

## Clean up<a name="scorekeep-ubuntu-cleanup"></a>

Run the `6-cleanup.sh` script to delete the bucket you created and take down your AWS CloudFormation stack\. You'll be asked to confirm with a **yes** or **no**\. After you confirm, you can exit the command line, terminate your Amazon EC2 instance, and delete your IAM role\.

**To terminate your Amazon EC2 instance \(console\)**

1. Sign in to the AWS Management Console and open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Verify you are in the Region where your instance was created\.

1. Choose **Instances**\.

1. In the list of instances, select **scorekeep\-ubuntu**\.

1. Choose **Actions**\.

1. From the list, choose **Instance State**, and then choose **Terminate**\.

1. To confirm, choose **Yes, Terminate**\.

**To delete your IAM role \(console\)**

1. Sign in to the AWS Management Console and open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. Choose **Roles**\. Search for **scorekeep\-ubuntu** and then select it\.

1. Choose **Delete role**, and then **Yes, delete**\.

## Next steps<a name="scorekeep-ubuntu-nextstep"></a>

Learn more about X\-Ray in [AWS X\-Ray concepts](xray-concepts.md)\.

To instrument your own app, learn more about the X\-Ray SDK for Java, or one of the other X\-Ray SDKs, see the following:
+ [AWS X\-Ray SDK for Java](xray-sdk-java.md)
+ [AWS X\-Ray SDK for Node\.js](xray-sdk-nodejs.md)
+ [AWS X\-Ray SDK for \.NET](xray-sdk-dotnet.md)

To run the X\-Ray daemon locally or on AWS, see [AWS X\-Ray daemon](xray-daemon.md)\.

To contribute to the sample application on GitHub, see [eb\-java\-scorekeep](https://github.com/awslabs/eb-java-scorekeep/tree/xray)\.