# Logging X\-Ray API calls with AWS CloudTrail<a name="xray-api-cloudtrail"></a>

AWS X\-Ray integrates with AWS CloudTrail to record API actions made by a user, a role, or an AWS service in X\-Ray\. You can use CloudTrail to monitor X\-Ray API requests in real time and store logs in Amazon S3, Amazon CloudWatch Logs, and Amazon CloudWatch Events\. X\-Ray supports logging the following actions as events in CloudTrail log files:

**Supported API Actions**
+ [https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html](https://docs.aws.amazon.com/xray/latest/api/API_PutEncryptionConfig.html)
+ [https://docs.aws.amazon.com/xray/latest/api/API_GetEncryptionConfig.html](https://docs.aws.amazon.com/xray/latest/api/API_GetEncryptionConfig.html)
+ [https://docs.aws.amazon.com/xray/latest/api/API_CreateGroup.html](https://docs.aws.amazon.com/xray/latest/api/API_CreateGroup.html)
+ [https://docs.aws.amazon.com/xray/latest/api/API_UpdateGroup.html](https://docs.aws.amazon.com/xray/latest/api/API_UpdateGroup.html)
+ [https://docs.aws.amazon.com/xray/latest/api/API_DeleteGroup.html](https://docs.aws.amazon.com/xray/latest/api/API_DeleteGroup.html)
+ [https://docs.aws.amazon.com/xray/latest/api/API_GetGroup.html](https://docs.aws.amazon.com/xray/latest/api/API_GetGroup.html)
+ [https://docs.aws.amazon.com/xray/latest/api/API_GetGroups.html](https://docs.aws.amazon.com/xray/latest/api/API_GetGroups.html)

**To create a trail**

1. Open the [**Trails** page of the CloudTrail console](https://console.aws.amazon.com/cloudtrail/home#/configuration)\.

1. Choose **Create trail**\.

1. Enter a trail name, and then choose the [types of event](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-and-data-events-with-cloudtrail.html) to record\.
   + **Management events** – Record API actions that create, read, update, or delete AWS resources\. Records calls to all supported API actions for all AWS services\.
   + **Data events** – Record API actions that target specific resources, like Amazon S3 object reads or AWS Lambda function invocations\. You choose which buckets and functions to monitor\.

1. Choose an Amazon S3 bucket and [encryption settings](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/encrypting-cloudtrail-log-files-with-aws-kms.html)\.

1. Choose **Create**\.

CloudTrail records API calls of the types you chose to log files in Amazon S3\. A CloudTrail log is an unordered array of events in JSON format\. For each call to a supported API action, CloudTrail records information about the request and the entity that made it\. Log events include the action name, parameters, the response from X\-Ray, and details about the requester\.

**Example X\-Ray GetEncryptionConfig log entry**  

```
{
    "eventVersion"=>"1.05",
    "userIdentity"=>{
        "type"=>"AssumedRole",
        "principalId"=>"AROAJVHBZWD3DN6CI2MHM:MyName",
        "arn"=>"arn:aws:sts::123456789012:assumed-role/MyRole/MyName",
        "accountId"=>"123456789012",
        "accessKeyId"=>"AKIAIOSFODNN7EXAMPLE",
        "sessionContext"=>{
            "attributes"=>{
                "mfaAuthenticated"=>"false",
                "creationDate"=>"2018-9-01T00:24:36Z"
            },
            "sessionIssuer"=>{
                "type"=>"Role",
                "principalId"=>"AROAJVHBZWD3DN6CI2MHM",
                "arn"=>"arn:aws:iam::123456789012:role/MyRole",
                "accountId"=>"123456789012",
                "userName"=>"MyRole"
            }
        }
    },
    "eventTime"=>"2018-9-01T00:24:36Z",
    "eventSource"=>"xray.amazonaws.com",
    "eventName"=>"GetEncryptionConfig",
    "awsRegion"=>"us-east-2",
    "sourceIPAddress"=>"33.255.33.255",
    "userAgent"=>"aws-sdk-ruby2/2.11.19 ruby/2.3.1 x86_64-linux",
    "requestParameters"=>nil,
    "responseElements"=>nil,
    "requestID"=>"3fda699a-32e7-4c20-37af-edc2be5acbdb",
    "eventID"=>"039c3d45-6baa-11e3-2f3e-e5a036343c9f",
    "eventType"=>"AwsApiCall",
    "recipientAccountId"=>"123456789012"
}
```

The [userIdentity element](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html) contains information about who generated the request\. The identity information helps you determine the following:
+ Whether the request was made with root or IAM user credentials\.
+ Whether the request was made with temporary security credentials for a role or federated user\.
+ Whether the request was made by another AWS service\.

To be notified when a new log file is available, configure CloudTrail to publish Amazon SNS notifications\. For more information, see [Configuring Amazon SNS Notifications for CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html)\.

You can also aggregate X\-Ray log files from multiple AWS Regions and multiple AWS accounts into a single Amazon S3 bucket\. For more information, see [Receiving CloudTrail Log Files from Multiple Regions](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html) and [Receiving CloudTrail Log Files from Multiple Accounts](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html)\.