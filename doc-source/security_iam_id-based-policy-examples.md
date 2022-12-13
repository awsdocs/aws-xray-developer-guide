# AWS X\-Ray identity\-based policy examples<a name="security_iam_id-based-policy-examples"></a>

By default, IAM users and roles don't have permission to create or modify X\-Ray resources\. They also can't perform tasks using the AWS Management Console, AWS CLI, or AWS API\. An IAM administrator must create IAM policies that grant users and roles permission to perform specific API operations on the specified resources they need\. The administrator must then attach those policies to the IAM users or groups that require those permissions\.

To learn how to create an IAM identity\-based policy using these example JSON policy documents, see [Creating Policies on the JSON Tab](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-json-editor) in the *IAM User Guide*\.

**Topics**
+ [Policy best practices](#security_iam_service-with-iam-policy-best-practices)
+ [Using the X\-Ray console](#security_iam_id-based-policy-examples-console)
+ [Allow users to view their own permissions](#security_iam_id-based-policy-examples-view-own-permissions)
+ [Managing access to X\-Ray groups and sampling rules based on tags](#security_iam_id-based-policy-examples-manage-sampling-tags)
+ [IAM managed policies for X\-Ray](#xray-permissions-managedpolicies)
+ [X\-Ray updates to AWS managed policies](#xray-permissions-managedpolicies-history)
+ [Specifying a resource within an IAM policy](#xray-permissions-resources)

## Policy best practices<a name="security_iam_service-with-iam-policy-best-practices"></a>

Identity\-based policies determine whether someone can create, access, or delete X\-Ray resources in your account\. These actions can incur costs for your AWS account\. When you create or edit identity\-based policies, follow these guidelines and recommendations:
+ **Get started with AWS managed policies and move toward least\-privilege permissions** – To get started granting permissions to your users and workloads, use the *AWS managed policies* that grant permissions for many common use cases\. They are available in your AWS account\. We recommend that you reduce permissions further by defining AWS customer managed policies that are specific to your use cases\. For more information, see [AWS managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) or [AWS managed policies for job functions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html) in the *IAM User Guide*\.
+ **Apply least\-privilege permissions** – When you set permissions with IAM policies, grant only the permissions required to perform a task\. You do this by defining the actions that can be taken on specific resources under specific conditions, also known as *least\-privilege permissions*\. For more information about using IAM to apply permissions, see [ Policies and permissions in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) in the *IAM User Guide*\.
+ **Use conditions in IAM policies to further restrict access** – You can add a condition to your policies to limit access to actions and resources\. For example, you can write a policy condition to specify that all requests must be sent using SSL\. You can also use conditions to grant access to service actions if they are used through a specific AWS service, such as AWS CloudFormation\. For more information, see [ IAM JSON policy elements: Condition](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) in the *IAM User Guide*\.
+ **Use IAM Access Analyzer to validate your IAM policies to ensure secure and functional permissions** – IAM Access Analyzer validates new and existing policies so that the policies adhere to the IAM policy language \(JSON\) and IAM best practices\. IAM Access Analyzer provides more than 100 policy checks and actionable recommendations to help you author secure and functional policies\. For more information, see [IAM Access Analyzer policy validation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html) in the *IAM User Guide*\.
+ **Require multi\-factor authentication \(MFA\)** – If you have a scenario that requires IAM users or root users in your account, turn on MFA for additional security\. To require MFA when API operations are called, add MFA conditions to your policies\. For more information, see [ Configuring MFA\-protected API access](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html) in the *IAM User Guide*\.

For more information about best practices in IAM, see [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) in the *IAM User Guide*\.

## Using the X\-Ray console<a name="security_iam_id-based-policy-examples-console"></a>

To access the AWS X\-Ray console, you must have a minimum set of permissions\. These permissions must allow you to list and view details about the X\-Ray resources in your AWS account\. If you create an identity\-based policy that is more restrictive than the minimum required permissions, the console won't function as intended for entities \(IAM users or roles\) with that policy\.

To ensure that those entities can still use the X\-Ray console, attach the `AWSXRayReadOnlyAccess` AWS managed policy to the entities\. This policy is described in more detail in [IAM managed policies for X\-Ray](#xray-permissions-managedpolicies)\. For more information, see [Adding Permissions to a User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html#users_change_permissions-add-console) in the *IAM User Guide*\. 

You don't need to allow minimum console permissions for users that are making calls only to the AWS CLI or the AWS API\. Instead, allow access to only the actions that match the API operation that you're trying to perform\.

## Allow users to view their own permissions<a name="security_iam_id-based-policy-examples-view-own-permissions"></a>

This example shows how you might create a policy that allows IAM users to view the inline and managed policies that are attached to their user identity\. This policy includes permissions to complete this action on the console or programmatically using the AWS CLI or AWS API\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ViewOwnUserInfo",
            "Effect": "Allow",
            "Action": [
                "iam:GetUserPolicy",
                "iam:ListGroupsForUser",
                "iam:ListAttachedUserPolicies",
                "iam:ListUserPolicies",
                "iam:GetUser"
            ],
            "Resource": ["arn:aws:iam::*:user/${aws:username}"]
        },
        {
            "Sid": "NavigateInConsole",
            "Effect": "Allow",
            "Action": [
                "iam:GetGroupPolicy",
                "iam:GetPolicyVersion",
                "iam:GetPolicy",
                "iam:ListAttachedGroupPolicies",
                "iam:ListGroupPolicies",
                "iam:ListPolicyVersions",
                "iam:ListPolicies",
                "iam:ListUsers"
            ],
            "Resource": "*"
        }
    ]
}
```

## Managing access to X\-Ray groups and sampling rules based on tags<a name="security_iam_id-based-policy-examples-manage-sampling-tags"></a>

You can use conditions in your identity\-based policy to control access to X\-Ray groups and sampling rules based on tags\. The following example policy could be used to deny an IAM user role the permissions to create, delete, or update groups with the tags `stage:prod` or `stage:preprod`\. For more information about tagging X\-Ray sampling rules and groups, see [Tagging X\-Ray sampling rules and groups](xray-tagging.md)\.

To deny a user access to create, update, or delete a group with a tag `stage:prod` or `stage:preprod`, assign the user a role with a policy similar to the following\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAllXRay",
            "Effect": "Allow",
            "Action": "xray:*",
            "Resource": "*"
        },
        {
            "Sid": "DenyCreateGroupWithStage",
            "Effect": "Deny",
            "Action": [
                "xray:CreateGroup"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/stage": [
                        "preprod",
                        "prod"
                    ]
                }
            }
        },
        {
            "Sid": "DenyUpdateGroupWithStage",
            "Effect": "Deny",
            "Action": [
                "xray:UpdateGroup",
                "xray:DeleteGroup"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/stage": [
                        "preprod",
                        "prod"
                    ]
                }
            }
        }
    ]
}
```

To deny the creation of a sampling rule, use `aws:RequestTag` to indicate tags that cannot be passed as part of a creation request\. To deny the update or deletion of a sampling rule, use `aws:ResourceTag` to deny actions based on the tags on those resources\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAllXRay",
            "Effect": "Allow",
            "Action": "xray:*",
            "Resource": "*"
        },
        {
            "Sid": "DenyCreateSamplingRuleWithStage",
            "Effect": "Deny",
            "Action": "xray:CreateSamplingRule",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/stage": [
                        "preprod",
                        "prod"
                    ]
                }
            }
        },
        {
            "Sid": "DenyUpdateSamplingRuleWithStage",
            "Effect": "Deny",
            "Action": [
                "xray:UpdateSamplingRule",
                "xray:DeleteSamplingRule"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/stage": [
                        "preprod",
                        "prod"
                    ]
                }
            }
        }
    ]
}
```

You can attach these policies \(or combine them into a single policy, then attach the policy\) to the IAM users in your account\. For the user to make changes to a group or sampling rule, the group or sampling rule must not be tagged `stage=prepod` or `stage=prod`\. The condition tag key `Stage` matches both `Stage` and `stage` because condition key names are not case\-sensitive\. For more information about the condition block, see [IAM JSON Policy Elements: Condition](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) in the *IAM User Guide*\.

A user with a role that has the following policy attached cannot add the tag `role:admin` to resources, and cannot remove tags from a resource that has `role:admin` associated with it\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAllXRay",
            "Effect": "Allow",
            "Action": "xray:*",
            "Resource": "*"
        },
        {
            "Sid": "DenyRequestTagAdmin",
            "Effect": "Deny",
            "Action": "xray:TagResource",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/role": "admin"
                }
            }
        },
        {
            "Sid": "DenyResourceTagAdmin",
            "Effect": "Deny",
            "Action": "xray:UntagResource",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/role": "admin"
                }
            }
        }
    ]
}
```

## IAM managed policies for X\-Ray<a name="xray-permissions-managedpolicies"></a>

To make granting permissions easy, IAM supports **managed policies** for each service\. A service can update these managed policies with new permissions when it releases new APIs\. AWS X\-Ray provides managed policies for read only, write only, and administrator use cases\.
+ `AWSXrayReadOnlyAccess` – Read permissions for using the X\-Ray console, AWS CLI, or AWS SDK to get trace data and service maps from the X\-Ray API\. Includes permission to view sampling rules\. The `BatchGetTraceSummaryById` and `GetDistinctTraceGraphs` API actions are not intended to be called by your code\. Therefore, these API actions are not included in the AWS CLI and AWS SDKs\.

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
                  "xray:BatchGetTraceSummaryById",
                  "xray:GetDistinctTraceGraphs",
                  "xray:GetServiceGraph",
                  "xray:GetTraceGraph",
                  "xray:GetTraceSummaries",
                  "xray:GetGroups",
                  "xray:GetGroup",
                  "xray:ListTagsForResource",
                  "xray:ListResourcePolicies",
                  "xray:GetTimeSeriesServiceStatistics",
                  "xray:GetInsightSummaries",
                  "xray:GetInsight",
                  "xray:GetInsightEvents",
                  "xray:GetInsightImpactGraph"
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

## X\-Ray updates to AWS managed policies<a name="xray-permissions-managedpolicies-history"></a>

View details about updates to AWS managed policies for X\-Ray since this service began tracking these changes\. For automatic alerts about changes to this page, subscribe to the RSS feed on the X\-Ray [Document history](document-history.md) page\.


| Change | Description | Date | 
| --- | --- | --- | 
|  [IAM managed policies for X\-Ray](#xray-permissions-managedpolicies) – Update to `AWSXrayReadOnlyAccess` policy\.  |  X\-Ray added an API action, `ListResourcePolicies`\.   | November 15, 2022 | 
|  [Using the X\-Ray console](#security_iam_id-based-policy-examples-console) – Update to `AWSXrayReadOnlyAccess` policy  |  X\-Ray added two new API actions, `BatchGetTraceSummaryById` and `GetDistinctTraceGraphs`\.  These actions are not intended to be called by your code\. Therefore, these API actions are not included in the AWS CLI and AWS SDKs\.  | November 11, 2022 | 

## Specifying a resource within an IAM policy<a name="xray-permissions-resources"></a>

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