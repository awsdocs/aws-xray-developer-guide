# AWS X\-Ray Permissions<a name="xray-permissions"></a>

You can use AWS Identity and Access Management \(IAM\) to grant X\-Ray permissions to users and compute resources in your account\. IAM controls access to the X\-Ray service at the API level to enforce permissions uniformly, regardless of which client \(console, AWS SDK, AWS CLI\) your users employ\.

To [use the X\-Ray console](xray-console.md) to view service maps and segments, you only need read permissions\. To enable console access, add the `AWSXrayReadOnlyAccess` [managed policy](#xray-permissions-managedpolicies) to your IAM user\.

For [local development and testing](#xray-permissions-local), create an IAM user with read and write permissions\. Generate access keys for the user and store them in the standard AWS SDK location\. You can use these credentials with the X\-Ray daemon, the AWS CLI, and the AWS SDK\.

To [deploy your instrumented app to AWS](#xray-permissions-aws), create an IAM role with write permissions and assign it to the resources running your application\. `AWSXRayDaemonWriteAccess` includes permission to upload traces, and some read permissions as well to support the use of [sampling rules](xray-console-sampling.md)\.

The read and write policies do not include permission to configure [encryption key settings](xray-console-encryption.md) and sampling rules\. Use `AWSXrayFullAccess` to access these settings, or add [configuration APIs](xray-api-configuration.md) in a custom policy\. For encryption and decryption with a customer managed key that you create, you also need [permission to use the key](#xray-permissions-encryption)\.

**Topics**
+ [IAM Managed Policies for X\-Ray](#xray-permissions-managedpolicies)
+ [Running Your Application Locally](#xray-permissions-local)
+ [Running Your Application in AWS](#xray-permissions-aws)
+ [User Permissions for Encryption](#xray-permissions-encryption)

## IAM Managed Policies for X\-Ray<a name="xray-permissions-managedpolicies"></a>

To make granting permissions easy, IAM supports **managed policies** for each service\. A service can update these managed policies with new permissions when it releases new APIs\. AWS X\-Ray provides [managed policies](#xray-permissions-managedpolicies) for read only, write only, and administrator use cases\.
+ `AWSXrayReadOnlyAccess` – Read permissions for using the X\-Ray console, AWS CLI, or AWS SDK to get trace data and service maps from the X\-Ray API\. Includes permission to view sampling rules\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "xray:GetSamplingRules",
                  "xray:GetSamplingTargets",
                  "xray:GetSamplingStatisticSummaries",
                  "xray:BatchGetTraces",
                  "xray:GetServiceGraph",
                  "xray:GetTraceGraph",
                  "xray:GetTraceSummaries"     
              ],
              "Resource": [
                  "*"
              ]
          }
      ]
  }
  ```
+ `AWSXRayDaemonWriteAccess` – Write permissions for using the X\-Ray daemon, AWS CLI, or AWS SDK to upload segment documents and telemetry to the X\-Ray API\. Includes read permissions to get [sampling rules](xray-console-sampling.md) and report sampling results\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "xray:PutTraceSegments",
                  "xray:PutTelemetryRecords",
                  "xray:GetSamplingRules",
                  "xray:GetSamplingTargets",
                  "xray:GetSamplingStatisticSummaries"
              ],
              "Resource": [
                  "*"
              ]
          }
      ]
  }
  ```
+ `AWSXrayFullAccess` – Permission to use all X\-Ray APIs, including read permissions, write permissions, and permission to configure encryption key settings and sampling rules\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "xray:*"
              ],
              "Resource": [
                  "*"
              ]
          }
      ]
  }
  ```

**To add a managed policy to an IAM user, group, or role**

1. Open the [IAM console](https://console.aws.amazon.com/iam/home)\.

1. Open the role associated with your instance profile, an IAM user, or an IAM group\.

1. Under **Permissions**, attach the managed policy\.

## Running Your Application Locally<a name="xray-permissions-local"></a>

Your instrumented application sends trace data to the X\-Ray daemon\. The daemon buffers segment documents and uploads them to the X\-Ray service in batches\. The daemon needs write permissions to upload trace data and telemetry to the X\-Ray service\.

When you [run the daemon locally](xray-daemon-local.md), store your IAM user's access key and secret key in a file named `credentials` in a folder named `.aws` in your user folder\.

**Example \~/\.aws/credentials**  

```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

If you already configured credentials for use with the AWS SDK or AWS CLI, the daemon can use those\. If multiple profiles are available, the daemon uses the default profile\.

## Running Your Application in AWS<a name="xray-permissions-aws"></a>

When you run your application on AWS, use a role to grant permission to the Amazon EC2 instance or Lambda function that runs the daemon\.
+ **Amazon Elastic Compute Cloud** – Create an IAM role and attach it to the EC2 instance as an [instance profile](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)\.
+ **Amazon Elastic Container Service** – Create an IAM role and attach it to container instances as a [container instance IAM role](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html)\.
+ **AWS Elastic Beanstalk** – Elastic Beanstalk includes X\-Ray permissions in its [default instance profile](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html#concepts-roles-instance)\. You can use the default instance profile, or add write permissions to a custom instance profile\.
+ **AWS Lambda** – Add write permissions to your function's execution role\.

**To create a role for use with X\-Ray**

1. Open the [IAM console](https://console.aws.amazon.com/iam/home)\.

1. Choose **Roles**\.

1. Choose **Create New Role**\.

1. For **Role Name**, type **xray\-application**\. Choose **Next Step\.**

1. For **Role Type**, choose **Amazon EC2**\.

1. Attach managed policies to give your application access to AWS services\.
   + **AWSXRayDaemonWriteAccess** – Gives the X\-Ray daemon permission to upload trace data\.
   + **AmazonS3ReadOnlyAccess** \(Amazon EC2 only\) – Gives the instance permission to download the X\-Ray daemon from Amazon S3\.

   If your application uses the AWS SDK to access other services, add policies that grant access to those services\.

1. Choose **Next Step**\.

1. Choose **Create Role**\.

## User Permissions for Encryption<a name="xray-permissions-encryption"></a>

X\-Ray encrypts all trace data and by default, and you can [configure it to use a key that you manage](xray-console-encryption.md)\. If you choose a AWS Key Management Service customer managed customer master key \(CMK\), you need to ensure that the key's access policy lets you grant permission to X\-Ray to use it to encrypt\. Other users in your account also need access to the key to view encrypted trace data in the X\-Ray console\.

For a customer managed CMK, configure your key with an access policy that allows the following actions\.
+ User who configures the key in X\-Ray has permission to call `kms:CreateGrant` and `kms:DescribeKey`\.
+ Users who access encrypted trace data have permission to call `kms:Decrypt`\.

When you add a user to the **Key users** group in the key configuration section of the AWS Identity and Access Management console, they have permission for both of these operations\. Permission only needs to be set on the key policy, so you don't need any AWS KMS permissions on your IAM users, groups, or roles\. See [Using Key Policies](http://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) in the AWS KMS Developer Guide for more information\.

For default encryption, or if you choose the AWS managed CMK \(`aws/xray`\), permission is based on who has access to X\-Ray APIs\. Anyone with access to [http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html), included in `AWSXrayFullAccess`, can change the encryption configuration\. To prevent a user from changing the encryption key, do not give them permission to use [http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html)\.