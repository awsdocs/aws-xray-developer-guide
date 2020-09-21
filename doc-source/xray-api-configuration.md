# Configuring sampling, groups, and encryption settings with the AWS X\-Ray API<a name="xray-api-configuration"></a>

AWS X\-Ray provides APIs for configuring [sampling rules](xray-console-sampling.md), group rules, and [encryption settings](xray-console-encryption.md)\.

**Topics**
+ [Encryption settings](#xray-api-configuration-encryption)
+ [Sampling rules](#xray-api-configuration-sampling)
+ [Groups](#xray-api-configuration-groups)

## Encryption settings<a name="xray-api-configuration-encryption"></a>

Use [https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html) to specify an AWS Key Management Service \(AWS KMS\) customer master key \(CMK\) to use for encryption\. 

**Note**  
X\-Ray does not support asymmetric CMKs\.

```
$ aws xray put-encryption-config --type KMS --key-id alias/aws/xray
{
    "EncryptionConfig": {
        "KeyId": "arn:aws:kms:us-east-2:123456789012:key/c234g4e8-39e9-4gb0-84e2-b0ea215cbba5",
        "Status": "UPDATING",
        "Type": "KMS"
    }
}
```

For the key ID, you can use an alias \(as shown in the example\), a key ID, or an Amazon Resource Name \(ARN\)\.

Use [https://docs.aws.amazon.com/xray/latest/api/API_GetEncryptionConfig.html](https://docs.aws.amazon.com/xray/latest/api/API_GetEncryptionConfig.html) to get the current configuration\. When X\-Ray finishes applying your settings, the status changes from `UPDATING` to `ACTIVE`\.

```
$ aws xray get-encryption-config
{
    "EncryptionConfig": {
        "KeyId": "arn:aws:kms:us-east-2:123456789012:key/c234g4e8-39e9-4gb0-84e2-b0ea215cbba5",
        "Status": "ACTIVE",
        "Type": "KMS"
    }
}
```

To stop using a CMK and use default encryption, set the encryption type to `NONE`\.

```
$ aws xray put-encryption-config --type NONE
{
    "EncryptionConfig": {
        "Status": "UPDATING",
        "Type": "NONE"
    }
}
```

## Sampling rules<a name="xray-api-configuration-sampling"></a>

You can manage the [sampling rules](xray-console-sampling.md) in your account with the X\-Ray API\. For more information about adding and managing tags, see [Tagging X\-Ray sampling rules and groups](xray-tagging.md)\.

Get all sampling rules with [https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html](https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html)\.

```
$ aws xray get-sampling-rules
{
    "SamplingRuleRecords": [
        {
            "SamplingRule": {
                "RuleName": "Default",
                "RuleARN": "arn:aws:xray:us-east-2:123456789012:sampling-rule/Default",
                "ResourceARN": "*",
                "Priority": 10000,
                "FixedRate": 0.05,
                "ReservoirSize": 1,
                "ServiceName": "*",
                "ServiceType": "*",
                "Host": "*",
                "HTTPMethod": "*",
                "URLPath": "*",
                "Version": 1,
                "Attributes": {}
            },
            "CreatedAt": 0.0,
            "ModifiedAt": 1529959993.0
        }
    ]
}
```

The default rule applies to all requests that don't match another rule\. It is the lowest priority rule and cannot be deleted\. You can, however, change the rate and reservoir size with [https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html)\.

**Example API input for [https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html) – 10000\-default\.json**  

```
{
    "SamplingRuleUpdate": {
        "RuleName": "Default",
        "FixedRate": 0.01,
        "ReservoirSize": 0
    }
}
```

The following example uses the previous file as input to change the default rule to one percent with no reservoir\. Tags are optional\. If you choose to add tags, a tag key is required, and tag values are optional\. To remove existing tags from a sampling rule, use [UntagResource](https://docs.aws.amazon.com/xray/latest/api/API_UntagResource.html)

```
$ aws xray update-sampling-rule --cli-input-json file://1000-default.json --tags [{"Key": "key_name","Value": "value"},{"Key": "key_name","Value": "value"}]
{
    "SamplingRuleRecords": [
        {
            "SamplingRule": {
                "RuleName": "Default",
                "RuleARN": "arn:aws:xray:us-east-2:123456789012:sampling-rule/Default",
                "ResourceARN": "*",
                "Priority": 10000,
                "FixedRate": 0.01,
                "ReservoirSize": 0,
                "ServiceName": "*",
                "ServiceType": "*",
                "Host": "*",
                "HTTPMethod": "*",
                "URLPath": "*",
                "Version": 1,
                "Attributes": {}
            },
            "CreatedAt": 0.0,
            "ModifiedAt": 1529959993.0
        },
```

Create additional sampling rules with [https://docs.aws.amazon.com/xray/latest/api/API_CreateSamplingRule.html](https://docs.aws.amazon.com/xray/latest/api/API_CreateSamplingRule.html)\. When you create a rule, most of the rule fields are required\. The following example creates two rules\. This first rule sets a base rate for the Scorekeep sample application\. It matches all requests served by the API that don't match a higher priority rule\.

**Example API input for [https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html) – 9000\-base\-scorekeep\.json**  

```
{
    "SamplingRule": {
        "RuleName": "base-scorekeep",
        "ResourceARN": "*",
        "Priority": 9000,
        "FixedRate": 0.1,
        "ReservoirSize": 5,
        "ServiceName": "Scorekeep",
        "ServiceType": "*",
        "Host": "*",
        "HTTPMethod": "*",
        "URLPath": "*",
        "Version": 1
    }
}
```

The second rule also applies to Scorekeep, but it has a higher priority and is more specific\. This rule sets a very low sampling rate for polling requests\. These are GET requests made by the client every few seconds to check for changes to the game state\.

**Example API input for [https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](https://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html) – 5000\-polling\-scorekeep\.json**  

```
{
    "SamplingRule": {
        "RuleName": "polling-scorekeep",
        "ResourceARN": "*",
        "Priority": 5000,
        "FixedRate": 0.003,
        "ReservoirSize": 0,
        "ServiceName": "Scorekeep",
        "ServiceType": "*",
        "Host": "*",
        "HTTPMethod": "GET",
        "URLPath": "/api/state/*",
        "Version": 1
    }
}
```

Tags are optional\. If you choose to add tags, a tag key is required, and tag values are optional\.

```
$ aws xray create-sampling-rule --cli-input-json file://5000-polling-scorekeep.json --tags [{"Key": "key_name","Value": "value"},{"Key": "key_name","Value": "value"}]
{
    "SamplingRuleRecord": {
        "SamplingRule": {
            "RuleName": "polling-scorekeep",
            "RuleARN": "arn:aws:xray:us-east-1:123456789012:sampling-rule/polling-scorekeep",
            "ResourceARN": "*",
            "Priority": 5000,
            "FixedRate": 0.003,
            "ReservoirSize": 0,
            "ServiceName": "Scorekeep",
            "ServiceType": "*",
            "Host": "*",
            "HTTPMethod": "GET",
            "URLPath": "/api/state/*",
            "Version": 1,
            "Attributes": {}
        },
        "CreatedAt": 1530574399.0,
        "ModifiedAt": 1530574399.0
    }
}
$ aws xray create-sampling-rule --cli-input-json file://9000-base-scorekeep.json
{
    "SamplingRuleRecord": {
        "SamplingRule": {
            "RuleName": "base-scorekeep",
            "RuleARN": "arn:aws:xray:us-east-1:123456789012:sampling-rule/base-scorekeep",
            "ResourceARN": "*",
            "Priority": 9000,
            "FixedRate": 0.1,
            "ReservoirSize": 5,
            "ServiceName": "Scorekeep",
            "ServiceType": "*",
            "Host": "*",
            "HTTPMethod": "*",
            "URLPath": "*",
            "Version": 1,
            "Attributes": {}
        },
        "CreatedAt": 1530574410.0,
        "ModifiedAt": 1530574410.0
    }
}
```

To delete a sampling rule, use [https://docs.aws.amazon.com/xray/latest/api/API_DeleteSamplingRule.html](https://docs.aws.amazon.com/xray/latest/api/API_DeleteSamplingRule.html)\.

```
$ aws xray delete-sampling-rule --rule-name polling-scorekeep
{
    "SamplingRuleRecord": {
        "SamplingRule": {
            "RuleName": "polling-scorekeep",
            "RuleARN": "arn:aws:xray:us-east-1:123456789012:sampling-rule/polling-scorekeep",
            "ResourceARN": "*",
            "Priority": 5000,
            "FixedRate": 0.003,
            "ReservoirSize": 0,
            "ServiceName": "Scorekeep",
            "ServiceType": "*",
            "Host": "*",
            "HTTPMethod": "GET",
            "URLPath": "/api/state/*",
            "Version": 1,
            "Attributes": {}
        },
        "CreatedAt": 1530574399.0,
        "ModifiedAt": 1530574399.0
    }
}
```

## Groups<a name="xray-api-configuration-groups"></a>

You can use the X\-Ray API to manage groups in your account\. Groups are a collection of traces that are defined by a filter expression\. You can use groups to generate additional service graphs and supply Amazon CloudWatch metrics\. See [Getting data from AWS X\-Ray](xray-api-gettingdata.md) for more details about working with service graphs and metrics through the X\-Ray API\. For more information about groups, see [Configuring groups in the X\-Ray console](xray-console-groups.md)\. For more information about adding and managing tags, see [Tagging X\-Ray sampling rules and groups](xray-tagging.md)\.

Create a group with `CreateGroup`\. Tags are optional\. If you choose to add tags, a tag key is required, and tag values are optional\.

```
$ aws xray create-group --group-name "TestGroup" --filter-expression "service(\"example.com\") {fault}" --tags [{"Key": "key_name","Value": "value"},{"Key": "key_name","Value": "value"}]
{
    "GroupName": "TestGroup",
    "GroupARN": "arn:aws:xray:us-east-2:123456789012:group/TestGroup/UniqueID",
    "FilterExpression": "service(\"example.com\") {fault OR error}"
}
```

Get all existing groups with `GetGroups`\.

```
$ aws xray get-groups
{
    "Groups": [
        {
            "GroupName": "TestGroup",
            "GroupARN": "arn:aws:xray:us-east-2:123456789012:group/TestGroup/UniqueID",
            "FilterExpression": "service(\"example.com\") {fault OR error}"
        },
		{
            "GroupName": "TestGroup2",
            "GroupARN": "arn:aws:xray:us-east-2:123456789012:group/TestGroup2/UniqueID",
            "FilterExpression": "responsetime > 2"
        }
    ],
	"NextToken": "tokenstring"
}
```

Update a group with `UpdateGroup`\. Tags are optional\. If you choose to add tags, a tag key is required, and tag values are optional\. To remove existing tags from a group, use [UntagResource](https://docs.aws.amazon.com/xray/latest/api/API_UntagResource.html)\.

```
$ aws xray update-group --group-name "TestGroup" --group-arn "arn:aws:xray:us-east-2:123456789012:group/TestGroup/UniqueID" --filter-expression "service(\"example.com\") {fault OR error}" --tags [{"Key": "Stage","Value": "Prod"},{"Key": "Department","Value": "QA"}]
{
    "GroupName": "TestGroup",
    "GroupARN": "arn:aws:xray:us-east-2:123456789012:group/TestGroup/UniqueID",
    "FilterExpression": "service(\"example.com\") {fault OR error}"
}
```

Delete a group with `DeleteGroup`\.

```
$ aws xray delete-group --group-name "TestGroup" --group-arn "arn:aws:xray:us-east-2:123456789012:group/TestGroup/UniqueID" 
    {
    }
```