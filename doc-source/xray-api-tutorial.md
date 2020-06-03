# Using the AWS X\-Ray API with the AWS CLI<a name="xray-api-tutorial"></a>

The AWS CLI lets your access the X\-Ray service directly and use the same APIs that the X\-Ray console uses to retrieve the service graph and raw traces data\. The sample application includes scripts that show how to use these APIs with the AWS CLI\.

## Prerequisites<a name="xray-api-tutorial-prerequisites"></a>

This tutorial uses the Scorekeep sample application and included scripts to generate tracing data and a service map\. Follow the instructions in the [getting started tutorial](xray-gettingstarted.md) to launch the application\.

This tutorial uses the AWS CLI to show basic use of the X\-Ray API\. The AWS CLI, [available for Windows, Linux, and OS\-X](https://docs.aws.amazon.com/cli/latest/userguide/installing.html), provides command line access to the public APIs for all AWS services\.

**Note**  
You must verify that your AWS CLI is configured to the same Region that your Scorekeep sample application was created in\.

Scripts included to test the sample application uses `cURL` to send traffic to the API and `jq` to parse the output\. You can download the `jq` executable from [stedolan\.github\.io](https://stedolan.github.io/jq/), and the `curl` executable from [https://curl.haxx.se/download.html](https://curl.haxx.se/download.html)\. Most Linux and OS X installations include cURL\.

## Generate trace data<a name="xray-api-tutorial-generatedata"></a>

The web app continues to generate traffic to the API every few seconds while the game is in\-progress, but only generates one type of request\. Use the `test-api.sh` script to run end to end scenarios and generate more diverse trace data while you test the API\.

**To use the `test-api.sh` script**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-console.html) for your environment\.

1. Copy the environment **URL** from the page header\.

1. Open `bin/test-api.sh` and replace the value for API with your environment's URL\.

   ```
   #!/bin/bash
   API=scorekeep.9hbtbm23t2.us-west-2.elasticbeanstalk.com/api
   ```

1. Run the script to generate traffic to the API\.

   ```
   ~/debugger-tutorial$ ./bin/test-api.sh
   Creating users,
   session,
   game,
   configuring game,
   playing game,
   ending game,
   game complete.
   {"id":"MTBP8BAS","session":"HUF6IT64","name":"tic-tac-toe-test","users":["QFF3HBGM","KL6JR98D"],"rules":"102","startTime":1476314241,"endTime":1476314245,"states":["JQVLEOM2","D67QLPIC","VF9BM9NC","OEAA6GK9","2A705O73","1U2LFTLJ","HUKIDD70","BAN1C8FI","G3UDJTUF","AB70HVEV"],"moves":["BS8F8LQ","4MTTSPKP","463OETES","SVEBCL3N","N7CQ1GHP","O84ONEPD","EG4BPROQ","V4BLIDJ3","9RL3NPMV"]}
   ```

## Use the X\-Ray API<a name="xray-api-tutorial-useapi"></a>

The AWS CLI provides commands for all of the API actions that X\-Ray provides, including [https://docs.aws.amazon.com/xray/latest/api/API_GetServiceGraph.html](https://docs.aws.amazon.com/xray/latest/api/API_GetServiceGraph.html) and [https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html](https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html)\. See the [AWS X\-Ray API Reference](https://docs.aws.amazon.com/xray/latest/api/Welcome.html) for more information on all of the supported actions and the data types that they use\.

**Example bin/service\-graph\.sh**  

```
EPOCH=$(date +%s)
aws xray get-service-graph --start-time $(($EPOCH-600)) --end-time $EPOCH
```
The script retrieves a service graph for the last 10 minutes\.  

```
~/eb-java-scorekeep$ ./bin/service-graph.sh | less
{
    "StartTime": 1479068648.0,
    "Services": [
        {
            "StartTime": 1479068648.0,
            "ReferenceId": 0,
            "State": "unknown",
            "EndTime": 1479068651.0,
            "Type": "client",
            "Edges": [
                {
                    "StartTime": 1479068648.0,
                    "ReferenceId": 1,
                    "SummaryStatistics": {
                        "ErrorStatistics": {
                            "ThrottleCount": 0,
                            "TotalCount": 0,
                            "OtherCount": 0
                        },
                        "FaultStatistics": {
                            "TotalCount": 0,
                            "OtherCount": 0
                        },
                        "TotalCount": 2,
                        "OkCount": 2,
                        "TotalResponseTime": 0.054000139236450195
                    },
                    "EndTime": 1479068651.0,
                    "Aliases": []
                }
            ]
        },
        {
            "StartTime": 1479068648.0,
            "Names": [
                "scorekeep.elasticbeanstalk.com"
            ],
            "ReferenceId": 1,
            "State": "active",
            "EndTime": 1479068651.0,
            "Root": true,
            "Name": "scorekeep.elasticbeanstalk.com",
...
```

**Example bin/trace\-urls\.sh**  

```
EPOCH=$(date +%s)
aws xray get-trace-summaries --start-time $(($EPOCH-120)) --end-time $(($EPOCH-60)) --query 'TraceSummaries[*].Http.HttpURL'
```
The script retrieves the URLs of traces generated between one and two minutes ago\.  

```
~/eb-java-scorekeep$ ./bin/trace-urls.sh
[
    "http://scorekeep.elasticbeanstalk.com/api/game/6Q0UE1DG/5FGLM9U3/endtime/1479069438",
    "http://scorekeep.elasticbeanstalk.com/api/session/KH4341QH",
    "http://scorekeep.elasticbeanstalk.com/api/game/GLQBJ3K5/153AHDIA",
    "http://scorekeep.elasticbeanstalk.com/api/game/VPDL672J/G2V41HM6/endtime/1479069466"
]
```

**Example bin/full\-traces\.sh**  

```
EPOCH=$(date +%s)
TRACEIDS=$(aws xray get-trace-summaries --start-time $(($EPOCH-120)) --end-time $(($EPOCH-60)) --query 'TraceSummaries[*].Id' --output text)
aws xray batch-get-traces --trace-ids $TRACEIDS --query 'Traces[*]'
```
The script retrieves full traces generated between one and two minutes ago\.  

```
~/eb-java-scorekeep$ ./bin/full-traces.sh | less
[
    {
        "Segments": [
            {
                "Id": "3f212bc237bafd5d",
                "Document": "{\"id\":\"3f212bc237bafd5d\",\"name\":\"DynamoDB\",\"trace_id\":\"1-5828d9f2-a90669393f4343211bc1cf75\",\"start_time\":1.479072242459E9,\"end_time\":1.479072242477E9,\"parent_id\":\"72a08dcf87991ca9\",\"http\":{\"response\":{\"content_length\":60,\"status\":200}},\"inferred\":true,\"aws\":{\"consistent_read\":false,\"table_name\":\"scorekeep-session-xray\",\"operation\":\"GetItem\",\"request_id\":\"QAKE0S8DD0LJM245KAOPMA746BVV4KQNSO5AEMVJF66Q9ASUAAJG\",\"resource_names\":[\"scorekeep-session-xray\"]},\"origin\":\"AWS::DynamoDB::Table\"}"
            },
            {
                "Id": "309e355f1148347f",
                "Document": "{\"id\":\"309e355f1148347f\",\"name\":\"DynamoDB\",\"trace_id\":\"1-5828d9f2-a90669393f4343211bc1cf75\",\"start_time\":1.479072242477E9,\"end_time\":1.479072242494E9,\"parent_id\":\"37f14ef837f00022\",\"http\":{\"response\":{\"content_length\":606,\"status\":200}},\"inferred\":true,\"aws\":{\"table_name\":\"scorekeep-game-xray\",\"operation\":\"UpdateItem\",\"request_id\":\"388GEROC4PCA6D59ED3CTI5EEJVV4KQNSO5AEMVJF66Q9ASUAAJG\",\"resource_names\":[\"scorekeep-game-xray\"]},\"origin\":\"AWS::DynamoDB::Table\"}"
            }
        ],
        "Id": "1-5828d9f2-a90669393f4343211bc1cf75",
        "Duration": 0.05099987983703613
    }
...
```

## Cleanup<a name="xray-api-tutorial-cleanup"></a>

Terminate your Elastic Beanstalk environment to shut down the Amazon EC2 instances, DynamoDB tables and other resources\.

**To terminate your Elastic Beanstalk environment**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-console.html) for your environment\.

1. Choose **Actions**\.

1. Choose **Terminate Environment**\.

1. Choose **Terminate**\.

Trace data is automatically deleted from X\-Ray after 30 days\.