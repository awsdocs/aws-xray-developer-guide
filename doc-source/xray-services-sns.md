# Amazon SNS and AWS X\-Ray<a name="xray-services-sns"></a>

AWS X\-Ray integrates with Amazon Simple Notification Service \(Amazon SNS\) to trace messages that are passed through Amazon SNS\. If an Amazon SNS publisher traces its client with the X\-Ray SDK, subscribers can retrieve the [tracing header](xray-concepts.md#xray-concepts-tracingheader) and continue to propagate the original trace from the publisher with the same trace ID\. This continuity enables users to trace, analyze, and debug throughout downstream services\. 

Amazon SNS trace context propagation currently supports the following subscribers:
+ **HTTP/HTTPS** – For HTTP/HTTPS subscribers, you can use the X\-Ray SDK to trace the incoming message request\. For more information and examples in Java, see [Tracing incoming requests with the X\-Ray SDK for Java](xray-sdk-java-filters.md)\.
+ **AWS Lambda** – For Lambda subscribers with active tracing enabled, Lambda records a segment with details about the function invocation, and sends it to the publisher's trace\. For more information, see [AWS Lambda and AWS X\-Ray](xray-services-lambda.md)\.

Use the following instructions to learn how to create a basic context between X\-Ray and Amazon SNS using a Lambda subscriber\. You will create two Lambda functions and an Amazon SNS topic\. Then, in the X\-Ray console, you can view the trace ID propagated throughout their interactions\.

## Requirements<a name="xray-services-sns-tutorial-requirements"></a>
+ [Node\.js 8 with `npm`](https://nodejs.org/en/download/releases/)
+ The Bash shell\. For Linux and macOS, this is included by default\. In Windows 10, you can install the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) to get a Windows\-integrated version of Ubuntu and Bash\.
+ [The AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

## Creating a Lambda subscriber function<a name="xray-services-sns-tutorial-subscriber"></a>

In the following steps, the sample Lambda function *MessageSubscriber* is implemented in Node\.js and is subscribed as an endpoint to an Amazon SNS topic\. The *MessageSubscriber* function creates a `custom process_message` subsegment with the [https://docs.aws.amazon.com/xray-sdk-for-nodejs/latest/reference/](https://docs.aws.amazon.com/xray-sdk-for-nodejs/latest/reference/) command\. The function writes a simple message as an annotation to the subsegment\.

**To create the subscriber package**

1. Create a file folder and name it to indicate it's the subscriber function \(for example, *sns\-xray\-subscriber*\)\. 

1. Create two files: `index.js` and `package.json`\.

1. Paste the following code into `index.js`\. 

   ```
   var AWSXRay = require('aws-xray-sdk');
   
   exports.handler = function(event, context, callback) {
       AWSXRay.captureFunc('process_message', function(subsegment) {
           var message = event.Records[0].Sns.Message;
           subsegment.addAnnotation('message_content', message);
           subsegment.close();
       });
       
       callback(null, "Message received.");
   };
   ```

1. Paste the following code into `package.json`\.

   ```
   {
     "name": "sns-xray-subscriber",
     "version": "1.0.0",
     "description": "A demo service to test SNS X-Ray trace header propagation",
     "dependencies": {
       "aws-xray-sdk": "^2.2.0"
     }
   }
   ```

1. Run the following script within the *sns\-xray\-subscriber* folder\. It creates a `package-lock.json` file and a `node_modules` folder, which handle all dependencies\.

   ```
   npm install --production
   ```

1. Compress the *sns\-xray\-subscriber* folder into a \.zip file\.

**To create a subscriber function and enable X\-Ray**

1. Open the [Lambda console](https://console.aws.amazon.com/lambda/home), and then choose **Create a function**\.

1.  Choose **Author from scratch**: 
   +  For **Function name**, provide a name \(for example, *MessageSubscriber*\)\. 
   +  For **Runtime**, use **Node\.js 10\.x**\. 

1. Choose **Create function** to create and display your function in the Lambda console\. 

1. In **Function code**, under **Code entry type**, choose **Upload a \.zip file**\.

1. Choose the subscriber package you created\. To upload, choose **Save** in the upper right of the console\.

1. In **Debugging and error handling**, select the **Enable AWS X\-Ray** box\.

1. Choose **Save**\.

## Creating an Amazon SNS topic<a name="xray-services-sns-tutorial-snstopic"></a>

When Amazon SNS receives requests, it propagates the trace header to its endpoint subscriber\. In the following steps, you create a topic and then set the endpoint as the Lambda function you created earlier\.

**To create an Amazon SNS topic and subscribe a Lambda function**

1. Open the [SNS console](https://console.aws.amazon.com/sns/home)\.

1. Choose **Topics**, and then choose **Create topic**\. For **Name**, provide a name\. 

1. In **Subscriptions**, choose **Create subscription**\. 

1. Record the topic ARN \(for example, `arn:aws:sns:{region}:{account id}:{topic name}`\)\.

1. Choose **Create subscription**:
   + For **Protocol**, choose **AWS Lambda**\.
   + For **Endpoint**, choose the ARN of the receiver Lambda function you created from the list of available Lambda functions\.

1. Choose **Create subscription**\.

## Creating a Lambda publisher function<a name="xray-services-sns-tutorial-publisher"></a>

In the following steps, the sample Lambda function *MessagePublisher* is implemented in Node\.js\. The function sends a message to the Amazon SNS topic you created earlier\. The function uses the AWS SDK for JavaScript to send notifications from Amazon SNS, and the X\-Ray SDK for Node\.js to instrument the AWS SDK client\.

**To create the publisher package**

1. Create a file folder and name it to indicate it's the publisher function \(for example, *sns\-xray\-publisher*\)\. 

1. Create two files: `index.js` and `package.json`\.

1. Paste the following code into `index.js`\.

   ```
   var AWSXRay = require('aws-xray-sdk');
   var AWS = AWSXRay.captureAWS(require('aws-sdk'));
   
   exports.handler = function(event, context, callback) {
       var sns = new AWS.SNS();
   
       sns.publish({
   	 // You can replace the following line with your custom message.
           Message: process.env.MESSAGE || "Testing X-Ray trace header propagation", 
           TopicArn: process.env.TOPIC_ARN
       }, function(err, data) {
           if (err) {
               console.log(err.stack);
               callback(err);
           } else {
               callback(null, "Message sent.");
           }
       });
   };
   ```

1. Paste the following code into `package.json`\.

   ```
   {
     "name": "sns-xray-publisher",
     "version": "1.0.0",
     "description": "A demo service to test SNS X-Ray trace header propagation",
     "dependencies": {
       "aws-xray-sdk": "^2.2.0"
     }
   }
   ```

1. Run the following script within the *sns\-xray\-publisher* folder\. It creates a `package-lock.json` file and a `node_modules` folder, which handle all dependencies\.

   ```
   npm install --production
   ```

1. Compress the *sns\-xray\-publisher* folder into a \.zip file\.

**To create a publisher function and enable X\-Ray**

1. Open the [Lambda console](https://console.aws.amazon.com/lambda/home), and then choose **Create function**\.

1. Choose **Author from scratch**: 
   +  For **Name**, provide a name \(for example, *MessagePublisher*\)\. 
   +  For **Runtime**, use **Node\.js 10\.x**\. 

1. In **Permissions**, expand **Choose or create an execution role**:
   + For **Execution role**, choose **Create a new role from AWS policy templates**\.
   + For **Role name**, provide a name\.
   + For **Policy templates**, choose **Amazon SNS publish policy**\.

1. Choose **Create function** to create and display your function in the Lambda console\. 

1. In **Function code**, under **Code entry type**, choose **Upload a \.zip file**\.

1. Choose the publisher package that you created\. To upload, choose **Save** in the upper right of the console\.

1. In **Environment variables**, add a variable:
   + For **Key**, use the key name `TOPIC_ARN`, which is defined in the publisher function\.
   + For **Value**, use the Amazon SNS topic ARN you recorded previously\.

1. Optionally, add another variable:
   + For **Key**, provide a key name \(for example, *MESSAGE*\)\.
   + For **Value**, enter any custom message\.

1. In **Debugging and error handling**, select the **Enable AWS X\-Ray** box\.

1. Choose **Save**\.

## Testing and validating context propagation<a name="xray-services-sns-tutorial-test"></a>

 Both publisher and subscriber functions enable Lambda active tracing when sending traces\. The publisher function uses the X\-Ray SDK to capture the `publish` SNS API call\. Then, Amazon SNS propagates the trace header to the subscriber\. Finally, the subscriber picks up the trace header and continues the trace\. Follow the trace ID in the following steps\.

**To create a publisher function and enable X\-Ray**

1. Open the [Lambda console](https://console.aws.amazon.com/lambda/home), and then choose the publisher function that you created previously\.

1. Choose **Test**\. 

1. Choose **Create a new test event**: 
   + For **Event template**, choose the **Hello World** template\.
   + For **Event name**, provide a name\.

1. Choose **Create**\.

1. Choose **Test** again\.

1. To verify, open the [X\-Ray console](https://console.aws.amazon.com/xray/home)\. Wait at least 10 seconds for the trace to appear\. 

1. When the service map is generated, validate that your two Lambda functions and the Amazon SNS topic appear\.

1. Choose the *MessageSubscriber* segment, and then choose **View traces**\.

1. Choose the trace from the list to reach the **Details** page\.

1. Choose the **process\_message** subsegment\.

1. Choose the **Annotations** tab to see the `message_content` key with the message value from the sender\.