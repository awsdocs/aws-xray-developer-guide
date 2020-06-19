# Instrumenting AWS Lambda functions<a name="scorekeep-lambda"></a>

Scorekeep uses two AWS Lambda functions\. The first is a Node\.js function from the `lambda` branch that generates random names for new users\. When a user creates a session without entering a name, the application calls a function named `random-name` with the AWS SDK for Java\. The X\-Ray SDK for Java records information about the call to Lambda in a subsegment like any other call made with an instrumented AWS SDK client\.

**Note**  
Running the `random-name` Lambda function requires the creation of additional resources outside of the Elastic Beanstalk environment\. See the readme for more information and instructions: [AWS Lambda Integration](https://github.com/awslabs/eb-java-scorekeep/tree/xray/README.md#aws-lambda-integration)\.

The second function, `scorekeep-worker`, is a Python function that runs independently of the Scorekeep API\. When a game ends, the API writes the session ID and game ID to an SQS queue\. The worker function reads items from the queue, and calls the Scorekeep API to construct complete records of each game session for storage in Amazon S3\.

Scorekeep includes AWS CloudFormation templates and scripts to create both functions\. Because you need to bundle the X\-Ray SDK with the function code, the templates create the functions without any code\. When you deploy Scorekeep, a configuration file included in the `.ebextensions` folder creates a source bundle that includes the SDK, and updates the function code and configuration with the AWS Command Line Interface\.

**Topics**
+ [Random name](#scorekeep-lambda-randomname)
+ [Worker](#scorekeep-lambda-worker)

## Random name<a name="scorekeep-lambda-randomname"></a>

Scorekeep calls the random name function when a user starts a game session without signing in or specifying a user name\. When Lambda processes the call to `random-name`, it reads the [tracing header](xray-concepts.md#xray-concepts-tracingheader), which contains the trace ID and sampling decision written by the X\-Ray SDK for Java\.

For each sampled request, Lambda runs the X\-Ray daemon and writes two segments\. The first segment records information about the call to Lambda that invokes the function\. This segment contains the same information as the subsegment recorded by Scorekeep, but from the Lambda point of view\. The second segment represents the work that the function does\.

Lambda passes the function segment to the X\-Ray SDK through the function context\. When you instrument a Lambda function, you don't use the SDK to [create a segment for incoming requests](xray-sdk-nodejs-middleware.md)\. Lambda provides the segment, and you use the SDK to instrument clients and write subsegments\.

![\[Service map showing how scorekeep calls a Lambda function to get random names for new users\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-lambda-node.png)

The `random-name` function is implemented in Node\.js\. It uses the SDK for JavaScript in Node\.js to send notifications with Amazon SNS, and the X\-Ray SDK for Node\.js to instrument the AWS SDK client\. To write annotations, the function creates a custom subsegment with `AWSXRay.captureFunc`, and writes annotations in the instrumented function\. In Lambda, you can't write annotations directly to the function segment, only to a subsegment that you create\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/function/index.js](https://github.com/awslabs/eb-java-scorekeep/tree/xray/function/index.js) \-\- Random name Lambda function**  

```
var AWSXRay = require('aws-xray-sdk-core');
var AWS = AWSXRay.captureAWS(require('aws-sdk'));

AWS.config.update({region: process.env.AWS_REGION});
var Chance = require('chance');

var myFunction = function(event, context, callback) {
  var sns = new AWS.SNS();
  var chance = new Chance();
  var userid = event.userid;
  var name = chance.first();

  AWSXRay.captureFunc('annotations', function(subsegment){
    subsegment.addAnnotation('Name', name);
    subsegment.addAnnotation('UserID', event.userid);
  });

  // Notify
  var params = {
    Message: 'Created randon name "' + name + '"" for user "' + userid + '".',
    Subject: 'New user: ' + name,
    TopicArn: process.env.TOPIC_ARN
  };
  sns.publish(params, function(err, data) {
    if (err) {
      console.log(err, err.stack);
      callback(err);
    }
    else {
      console.log(data);
      callback(null, {"name": name});
    }
  });
};

exports.handler = myFunction;
```

This function is created automatically when you deploy the sample application to Elastic Beanstalk\. The `xray` branch includes a script to create a blank Lambda function\. Configuration files in the `.ebextensions` folder build the function package with `npm install` during deployment, and then update the Lambda function with the AWS CLI\.

## Worker<a name="scorekeep-lambda-worker"></a>

The instrumented worker function is provided in its own branch, `xray-worker`, as it cannot run unless you create the worker function and related resources first\. See [the branch readme](https://github.com/awslabs/eb-java-scorekeep/tree/xray-worker/README.md) for instructions\.

The function is triggered by a bundled Amazon CloudWatch Events event every 5 minutes\. When it runs, the function pulls an item from an Amazon SQS queue that Scorekeep manages\. Each message contains information about a completed game\.

The worker pulls the game record and documents from other tables that the game record references\. For example, the game record in DynamoDB includes a list of moves that were executed during the game\. The list does not contain the moves themselves, but rather IDs of moves that are stored in a separate table\.

Sessions, and states are stored as references as well\. This keeps the entries in the game table from being too large, but requires additional calls to get all of the information about the game\. The worker dereferences all of these entries and constructs a complete record of the game as a single document in Amazon S3\. When you want to do analytics on the data, you can run queries on it directly in Amazon S3 with Amazon Athena without running read\-heavy data migrations to get your data out of DynamoDB\.

![\[Service map showing how the scorekeep worker function uses Amazon SQS, Amazon S3, and the scorekeep API.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-lambdaworker-node.png)

The worker function has active tracing enabled in its configuration in AWS Lambda\. Unlike the random name function, the worker does not receive a request from an instrumented application, so AWS Lambda doesn't receive a tracing header\. With active tracing, Lambda creates the trace ID and makes sampling decisions\.

The X\-Ray SDK for Python is just a few lines at the top of the function that import the SDK and run its `patch_all` function to patch the AWS SDK for Python \(Boto\) and HTTclients that it uses to call Amazon SQS and Amazon S3\. When the worker calls the Scorekeep API, the SDK adds the [tracing header](xray-concepts.md#xray-concepts-tracingheader) to the request to trace calls through the API\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray-worker/_lambda/scorekeep-worker/scorekeep-worker.py](https://github.com/awslabs/eb-java-scorekeep/tree/xray-worker/_lambda/scorekeep-worker/scorekeep-worker.py) \-\- Worker Lambda function**  

```
import os
import boto3
import json
import requests
import time
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()
queue_url = os.environ['WORKER_QUEUE']

def lambda_handler(event, context):
    # Create SQS client
    sqs = boto3.client('sqs')
    s3client = boto3.client('s3')

    # Receive message from SQS queue
    response = sqs.receive_message(
        QueueUrl=queue_url,
        AttributeNames=[
            'SentTimestamp'
        ],
        MaxNumberOfMessages=1,
        MessageAttributeNames=[
            'All'
        ],
        VisibilityTimeout=0,
        WaitTimeSeconds=0
    )
   ...
```