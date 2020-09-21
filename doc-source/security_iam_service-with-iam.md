# How AWS X\-Ray works with IAM<a name="security_iam_service-with-iam"></a>

Before you use IAM to manage access to X\-Ray, you should understand what IAM features are available to use with X\-Ray\. To get a high\-level view of how X\-Ray and other AWS services work with IAM, see [AWS Services That Work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) in the *IAM User Guide*\.

You can use AWS Identity and Access Management \(IAM\) to grant X\-Ray permissions to users and compute resources in your account\. IAM controls access to the X\-Ray service at the API level to enforce permissions uniformly, regardless of which client \(console, AWS SDK, AWS CLI\) your users employ\.

To [use the X\-Ray console](xray-console.md) to view service maps and segments, you only need read permissions\. To enable console access, add the `AWSXrayReadOnlyAccess` [managed policy](security_iam_id-based-policy-examples.md#xray-permissions-managedpolicies) to your IAM user\.

For [local development and testing](#xray-permissions-local), create an IAM user with read and write permissions\. Generate access keys for the user and store them in the standard AWS SDK location\. You can use these credentials with the X\-Ray daemon, the AWS CLI, and the AWS SDK\.

To [deploy your instrumented app to AWS](#xray-permissions-aws), create an IAM role with write permissions and assign it to the resources running your application\. `AWSXRayDaemonWriteAccess` includes permission to upload traces, and some read permissions as well to support the use of [sampling rules](xray-console-sampling.md)\.

The read and write policies do not include permission to configure [encryption key settings](xray-console-encryption.md) and sampling rules\. Use `AWSXrayFullAccess` to access these settings, or add [configuration APIs](xray-api-configuration.md) in a custom policy\. For encryption and decryption with a customer managed key that you create, you also need [permission to use the key](#xray-permissions-encryption)\.

**Topics**
+ [X\-Ray identity\-based policies](#security_iam_service-with-iam-id-based-policies)
+ [X\-Ray resource\-based policies](#security_iam_service-with-iam-resource-based-policies)
+ [Authorization based on X\-Ray tags](#security_iam_service-with-iam-tags)
+ [Running your application locally](#xray-permissions-local)
+ [Running your application in AWS](#xray-permissions-aws)
+ [User permissions for encryption](#xray-permissions-encryption)

## X\-Ray identity\-based policies<a name="security_iam_service-with-iam-id-based-policies"></a>

With IAM identity\-based policies, you can specify allowed or denied actions and resources as well as the conditions under which actions are allowed or denied\. X\-Ray supports specific actions, resources, and condition keys\. To learn about all of the elements that you use in a JSON policy, see [IAM JSON Policy Elements Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) in the *IAM User Guide*\.

### Actions<a name="security_iam_service-with-iam-id-based-policies-actions"></a>

The `Action` element of an IAM identity\-based policy describes the specific action or actions that will be allowed or denied by the policy\. Policy actions usually have the same name as the associated AWS API operation\. The action is used in a policy to grant permissions to perform the associated operation\. 

Policy actions in X\-Ray use the following prefix before the action: `xray:`\. For example, to grant someone permission to retrieve group resource details with the X\-Ray `GetGroup` API operation, you include the `xray:GetGroup` action in their policy\. Policy statements must include either an `Action` or `NotAction` element\. X\-Ray defines its own set of actions that describe tasks that you can perform with this service\.

To specify multiple actions in a single statement, separate them with commas as follows:

```
"Action": [
      "xray:action1",
      "xray:action2"
```

You can specify multiple actions using wildcards \(\*\)\. For example, to specify all actions that begin with the word `Get`, include the following action:

```
"Action": "xray:Get*"
```

To see a list of X\-Ray actions, see [Actions Defined by AWS X\-Ray](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsx-ray.html) in the *IAM User Guide*\.

### Resources<a name="security_iam_service-with-iam-id-based-policies-resources"></a>

The `Resource` element specifies the object or objects to which the action applies\. Statements must include either a `Resource` or a `NotResource` element\. You specify a resource using an ARN or using the wildcard \(\*\) to indicate that the statement applies to all resources\.

You can control access to resources by using an IAM policy\. For actions that support resource\-level permissions, you use an Amazon Resource Name \(ARN\) to identify the resource that the policy applies to\.

All X\-Ray actions can be used in an IAM policy to grant or deny users permission to use that action\. However, not all [X\-Ray actions](https://docs.aws.amazon.com/xray/latest/api/API_Operations.html) support resource\-level permissions, which enable you to specify the resources on which an action can be performed\.

For actions that don't support resource\-level permissions, you must use "`*`" as the resource\.

The following X\-Ray actions support resource\-level permissions:
+ `CreateGroup`
+ `GetGroup`
+ `UpdateGroup`
+ `DeleteGroup`
+ `CreateSamplingRule`
+ `UpdateSamplingRule`
+ `DeleteSamplingRule`

The following is an example of an identity\-based permissions policy for a `CreateGroup` action\. The example shows the use of an ARN relating to Group name `local-users` with the unique ID as a wildcard\. The unique ID is generated when the group is created, and so it can't be predicted in the policy in advance\. When using `GetGroup`, `UpdateGroup`, or `DeleteGroup`, you can define this as either a wildcard or the exact ARN, including ID\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "xray:CreateGroup"
            ],
            "Resource": [
                "arn:aws:xray:eu-west-1:123456789012:group/local-users/*"
            ]
        }
    ]
}
```

The following is an example of an identity\-based permissions policy for a `CreateSamplingRule` action\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "xray:CreateSamplingRule"
            ],
            "Resource": [
                "arn:aws:xray:eu-west-1:123456789012:sampling-rule/base-scorekeep"
            ]
        }
    ]
}
```

**Note**  
The ARN of a sampling rule is defined by its name\. Unlike group ARNs, sampling rules have no uniquely generated ID\.

To see a list of X\-Ray resource types and their ARNs, see [Resources Defined by AWS X\-Ray](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awskeymanagementservice.html#awskeymanagementservice-resources-for-iam-policies) in the *IAM User Guide*\. To learn with which actions you can specify the ARN of each resource, see [Actions Defined by AWS X\-Ray](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsx-ray.html)\.

### Condition keys<a name="security_iam_service-with-iam-id-based-policies-conditionkeys"></a>

X\-Ray does not provide any service\-specific condition keys, but it does support using some global condition keys\. To see all AWS global condition keys, see [AWS Global Condition Context Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

### Examples<a name="security_iam_service-with-iam-id-based-policies-examples"></a>

To view examples of X\-Ray identity\-based policies, see [AWS X\-Ray identity\-based policy examples](security_iam_id-based-policy-examples.md)\.

## X\-Ray resource\-based policies<a name="security_iam_service-with-iam-resource-based-policies"></a>

X\-Ray does not support resource\-based policies, but it does support resource\-level policies\.

## Authorization based on X\-Ray tags<a name="security_iam_service-with-iam-tags"></a>

You can attach tags to X\-Ray groups or sampling rules, or pass tags in a request to X\-Ray\. To control access based on tags, you provide tag information in the [condition element](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) of a policy using the `xray:ResourceTag/key-name`, `aws:RequestTag/key-name`, or `aws:TagKeys` condition keys\. For more information about tagging X\-Ray resources, see [Tagging X\-Ray sampling rules and groups](xray-tagging.md)\.

To view an example identity\-based policy for limiting access to a resource based on the tags on that resource, see [Managing access to X\-Ray groups and sampling rules based on tags](security_iam_id-based-policy-examples.md#security_iam_id-based-policy-examples-manage-sampling-tags)\.

## Running your application locally<a name="xray-permissions-local"></a>

Your instrumented application sends trace data to the X\-Ray daemon\. The daemon buffers segment documents and uploads them to the X\-Ray service in batches\. The daemon needs write permissions to upload trace data and telemetry to the X\-Ray service\.

When you [run the daemon locally](xray-daemon-local.md), store your IAM user's access key and secret key in a file named `credentials` in a folder named `.aws` in your user folder\.

**Example \~/\.aws/credentials**  

```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

If you already configured credentials for use with the AWS SDK or AWS CLI, the daemon can use those\. If multiple profiles are available, the daemon uses the default profile\.

## Running your application in AWS<a name="xray-permissions-aws"></a>

When you run your application on AWS, use a role to grant permission to the Amazon EC2 instance or Lambda function that runs the daemon\.
+ **Amazon Elastic Compute Cloud \(Amazon EC2\)** – Create an IAM role and attach it to the EC2 instance as an [instance profile](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)\.
+ **Amazon Elastic Container Service \(Amazon ECS\)** – Create an IAM role and attach it to container instances as a [container instance IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html)\.
+ **AWS Elastic Beanstalk \(Elastic Beanstalk\)** – Elastic Beanstalk includes X\-Ray permissions in its [default instance profile](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html#concepts-roles-instance)\. You can use the default instance profile, or add write permissions to a custom instance profile\.
+ **AWS Lambda \(Lambda\)** – Add write permissions to your function's execution role\.

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

## User permissions for encryption<a name="xray-permissions-encryption"></a>

X\-Ray encrypts all trace data and by default, and you can [configure it to use a key that you manage](xray-console-encryption.md)\. If you choose a AWS Key Management Service customer managed customer master key \(CMK\), you need to ensure that the key's access policy lets you grant permission to X\-Ray to use it to encrypt\. Other users in your account also need access to the key to view encrypted trace data in the X\-Ray console\.

For a customer managed CMK, configure your key with an access policy that allows the following actions:
+ User who configures the key in X\-Ray has permission to call `kms:CreateGrant` and `kms:DescribeKey`\.
+ Users who can access encrypted trace data have permission to call `kms:Decrypt`\.

When you add a user to the **Key users** group in the key configuration section of the IAM console, they have permission for both of these operations\. Permission only needs to be set on the key policy, so you don't need any AWS KMS permissions on your IAM users, groups, or roles\. For more information, see [Using Key Policies in the AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)\.

For default encryption, or if you choose the AWS managed CMK \(`aws/xray`\), permission is based on who has access to X\-Ray APIs\. Anyone with access to [https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html), included in `AWSXrayFullAccess`, can change the encryption configuration\. To prevent a user from changing the encryption key, do not give them permission to use [https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html)\.