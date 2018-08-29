# Configuring AWS X\-Ray Sampling and Encryption Settings with the API<a name="xray-api-configuration"></a>

AWS X\-Ray provides APIs for configuring [sampling rules](xray-console-sampling.md) and [encryption settings](xray-console-encryption.md)\.

**Topics**
+ [Encryption Settings](#xray-api-configuration-encryption)
+ [Sampling Rules](#xray-api-configuration-sampling)

## Encryption Settings<a name="xray-api-configuration-encryption"></a>

Use [http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html) to specify a customer master key \(CMK\) to use for encryption\.

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

Use [http://docs.aws.amazon.com/xray/latest/api/API_GetEncryptionConfig.html](http://docs.aws.amazon.com/xray/latest/api/API_GetEncryptionConfig.html) to get the current configuration\. When X\-Ray finishes applying your settings, the status changes from `UPDATING` to `ACTIVE`\.

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

## Sampling Rules<a name="xray-api-configuration-sampling"></a>

You can manage the [sampling rules](xray-console-sampling.md) in your account with the X\-Ray API\.

Get all sampling rules with [http://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html](http://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html)\.

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

The default rule applies to all requests that don't match another rule\. It is the lowest priority rule and cannot be deleted\. You can, however, change the rate and reservoir size with [http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html)\.

**Example API input for [http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html) – 10000\-default\.json**  

```
{
    "SamplingRuleUpdate": {
        "RuleName": "Default",
        "FixedRate": 0.01,
        "ReservoirSize": 0
    }
}
```

The following example uses the previous file as input to change the default rule to one percent with no reservoir\.

```
$ aws xray update-sampling-rule --cli-input-json file://1000-default.json
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

Create additional sampling rules with [http://docs.aws.amazon.com/xray/latest/api/API_CreateSamplingRule.html](http://docs.aws.amazon.com/xray/latest/api/API_CreateSamplingRule.html)\. When you create a rule, most of the rule fields are required\. The following example creates two rules\. This first rule sets a base rate for the Scorekeep sample application\. It matches all requests served by the API that don't match a higher priority rule\.

**Example API input for [http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html) – 9000\-base\-scorekeep\.json**  

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

**Example API input for [http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html](http://docs.aws.amazon.com/xray/latest/api/API_UpdateSamplingRule.html) – 5000\-polling\-scorekeep\.json**  

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

```
$ aws xray create-sampling-rule --cli-input-json file://5000-polling-scorekeep.json
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

To delete a sampling rule, use [http://docs.aws.amazon.com/xray/latest/api/API_DeleteSamplingRule.html](http://docs.aws.amazon.com/xray/latest/api/API_DeleteSamplingRule.html)\.

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