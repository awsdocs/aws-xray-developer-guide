# Configuring Encryption Settings in the AWS X\-Ray Console<a name="xray-console-encryption"></a>

AWS X\-Ray always encrypts traces and related data at rest\. If you need to audit and disable encryption keys for compliance or internal requirements, you can configure X\-Ray to use an AWS Key Management Service \(AWS KMS\) customer master key \(CMK\) to encrypt data\.

X\-Ray provides an AWS managed CMK named `aws/xray`\. Use this key if you just want to [audit key usage in AWS CloudTrail](http://docs.aws.amazon.com/kms/latest/developerguide/logging-using-cloudtrail.html) and don't need to manage the key itself\. If you need to manage access to the key or configure key rotation, you can [create a customer managed CMK](http://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html)\.

**To configure X\-Ray to use a CMK for encryption**

1. Open the [X\-Ray console](https://console.aws.amazon.com/xray/home#/service-map)\.

1. Choose **Encryption**\.

1. Choose **Use a customer master key**\.

1. Choose a key from the drop\-down menu:
   + **aws/xray** – Use the AWS managed CMK\.
   + *key alias* – Use a customer managed CMK in your account\.
   + **Manually enter a key ARN** – Use a customer managed CMK in a different account\. Enter the full Amazon Resource Name \(ARN\) of the key in the field that appears\.

1. Choose **Apply**\.

When you change encryption settings, X\-Ray spends some time generating and propagating data keys\. While the new key is being processed, X\-Ray may encrypt data with a combination of the new and old settings\. Existing data is not re\-encrypted when you change encryption settings\.

**Note**  
AWS KMS charges when X\-Ray uses a CMK to encrypt or decrypt trace data\.  
**Default encryption** – Free\.
**AWS managed CMK** – Pay for key use\.
**Customer managed CMK** – Pay for key storage and use\.
See [AWS Key Management Service Pricing](https://aws.amazon.com/kms/pricing/) for details\.

If X\-Ray is unable to access your encryption key, it stops storing data\. This can happen if your user loses access to the CMK, or if you disable a key that's currently in use\. When this happens, X\-Ray shows a notification in the navigation bar\.

To configure encryption settings with the X\-Ray API, see [Configuring AWS X\-Ray Sampling and Encryption Settings with the API](xray-api-configuration.md)\.