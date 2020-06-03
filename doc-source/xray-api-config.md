# Tracking X\-Ray encryption configuration changes with AWS Config<a name="xray-api-config"></a>

AWS X\-Ray integrates with AWS Config to record configuration changes made to your X\-Ray encryption resources\. You can use AWS Config to inventory X\-Ray encryption resources, audit the X\-Ray configuration history, and send notifications based on resource changes\.

AWS Config supports logging the following X\-Ray encryption resource changes as events:
+ **Configuration changes** – Changing or adding an encryption key, or reverting to the default X\-Ray encryption setting\.

Use the following instructions to learn how to create a basic connection between X\-Ray and AWS Config\. 

## Creating a Lambda function trigger<a name="LambdaFunctionTrigger"></a>

You must have the ARN of a custom AWS Lambda function before you can generate a custom AWS Config rule\. Follow these instructions to create a basic function with Node\.js that returns a compliant or non\-compliant value back to AWS Config based on the state of the `XrayEncryptionConfig` resource\.

**To create a Lambda function with an AWS::XrayEncryptionConfig change trigger**

1. Open the [Lambda console](https://console.aws.amazon.com/lambda/home)\. Choose **Create function**\.

1. Choose **Blueprints**, and then filter the blueprints library for the **config\-rule\-change\-triggered** blueprint\. Either click the link in the blueprint's name or choose **Configure** to continue\.

1. Define the following fields to configure the blueprint:
   + For **Name**, type a name\.
   + For **Role**, choose **Create new role from template\(s\)**\.
   + For **Role name**, type a name\.
   + For **Policy templates**, choose **AWS Config Rules permissions**\.

1. Choose **Create function** to create and display your function in the AWS Lambda console\.

1. Edit your function code to replace `AWS::EC2::Instance` with `AWS::XrayEncryptionConfig`\. You can also update the description field to reflect this change\.

   **Default Code**

   ```
       if (configurationItem.resourceType !== 'AWS::EC2::Instance') {
           return 'NOT_APPLICABLE';
       } else if (ruleParameters.desiredInstanceType === configurationItem.configuration.instanceType) {
           return 'COMPLIANT';
       }
           return 'NON_COMPLIANT';
   ```

   **Updated Code**

   ```
       if (configurationItem.resourceType !== 'AWS::XRay::EncryptionConfig') {
           return 'NOT_APPLICABLE';
       } else if (ruleParameters.desiredInstanceType === configurationItem.configuration.instanceType) {
           return 'COMPLIANT';
       }
           return 'NON_COMPLIANT';
   ```

1. Add the following to your execution role in IAM for access to X\-Ray\. These permissions allow read\-only access to your X\-Ray resources\. Failure to provide access to the appropriate resources will result in an out of scope message from AWS Config when it evaluates the Lambda function associated with the rule\.

   ```
       {
           "Sid": "Stmt1529350291539",
           "Action": [
               "xray:GetEncryptionConfig"
           ],
           "Effect": "Allow",
           "Resource": "*"
        }
   ```

## Creating a custom AWS Config rule for x\-ray<a name="ConfigRule"></a>

When the Lambda function is created, note the function's ARN, and go to the AWS Config console to create your custom rule\. 

**To create an AWS Config rule for X\-Ray**

1. Open the [**Rules** page of the AWS Config console](https://console.aws.amazon.com/config/home#/rules/view)\.

1. Choose **Add rule**, and then choose **Add custom rule**\.

1. In **AWS Lambda Function ARN**, insert the ARN associated with the Lambda function you want to use\.

1. Choose the type of trigger to set:
   + **Configuration changes** – AWS Config triggers the evaluation when any resource that matches the rule's scope changes in configuration\. The evaluation runs after AWS Config sends a configuration item change notification\.
   + **Periodic** – AWS Config runs evaluations for the rule at a frequency that you choose \(for example, every 24 hours\)\.

1. For **Resource type**, choose **EncryptionConfig** in the X\-Ray section\.

1. Choose ****Save****\.

The AWS Config console begins to evaluate the rule's compliance immediately\. The evaluation can take several minutes to complete\.

Now that this rule is compliant, AWS Config can begin to compile an audit history\. AWS Config records resource changes in the form of a timeline\. For each change in the timeline of events, AWS Config generates a table in a from/to format to show what changed in the JSON representation of the encryption key\. The two field changes associated with EncryptionConfig are `Configuration.type` and `Configuration.keyID`\.

## Example results<a name="Examples"></a>

Following is an example of an AWS Config timeline showing changes made at specific dates and times\.

![\[AWS Config timeline\]](http://docs.aws.amazon.com/xray/latest/devguide/images/ConfigTimeline.png)

Following is an example of an AWS Config change entry\. The from/to format illustrates what changed\. This example shows that the default X\-Ray encryption settings were changed to a defined encryption key\.

![\[X-Ray encryption configuration change entry\]](http://docs.aws.amazon.com/xray/latest/devguide/images/ConfigChanges.png)

## Amazon SNS notifications<a name="SNSNotifs"></a>

To be notified of configuration changes, set AWS Config to publish Amazon SNS notifications\. For more information, see [Monitoring AWS Config Resource Changes by Email](https://docs.aws.amazon.com/config/latest/developerguide/monitoring-resource-changes-by-email.html)\.