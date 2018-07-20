# Configuring AWS X\-Ray Encryption Settings<a name="xray-api-configuration"></a>

AWS X\-Ray provides APIs for configuring [encryption settings](xray-console-encryption.md)\. Use [http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](http://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html) to specify a customer master key \(CMK\) to use for encryption\.

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

For the key ID, you can use an alias \(as shown in the example\), a key ID, or an ARN\.

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