# Sending trace data to AWS X\-Ray<a name="xray-api-sendingdata"></a>

You can send trace data to X\-Ray in the form of segment documents\. A segment document is a JSON formatted string that contains information about the work that your application does in service of a request\. Your application can record data about the work that it does itself in segments, or work that uses downstream services and resources in subsegments\.

Segments record information about the work that your application does\. A segment, at a minimum, records the time spent on a task, a name, and two IDs\. The trace ID tracks the request as it travels between services\. The segment ID tracks the work done for the request by a single service\.

**Example Minimal complete segment**  

```
{
  "name" : "Scorekeep",
  "id" : "70de5b6f19ff9a0a",
  "start_time" : 1.478293361271E9,
  "trace_id" : "1-581cf771-a006649127e371903a2de979",
  "end_time" : 1.478293361449E9
}
```

When a request is received, you can send an in\-progress segment as a placeholder until the request is completed\.

**Example In\-progress segment**  

```
{
  "name" : "Scorekeep",
  "id" : "70de5b6f19ff9a0b",
  "start_time" : 1.478293361271E9,
  "trace_id" : "1-581cf771-a006649127e371903a2de979",
  “in_progress”: true
}
```

You can send segments to X\-Ray directly, with [`PutTraceSegments`](#xray-api-segments), or [through the X\-Ray daemon](#xray-api-daemon)\.

Most applications call other services or access resources with the AWS SDK\. Record information about downstream calls in *subsegments*\. X\-Ray uses subsegments to identify downstream services that don't send segments and create entries for them on the service graph\.

A subsegment can be embedded in a full segment document, or sent separately\. Send subsegments separately to asynchronously trace downstream calls for long\-running requests, or to avoid exceeding the maximum segment document size \(64 kB\)\.

**Example Subsegment**  
A subsegment has a `type` of `subsegment` and a `parent_id` that identifies the parent segment\.  

```
{
  "name" : "www2.example.com",
  "id" : "70de5b6f19ff9a0c",
  "start_time" : 1.478293361271E9,
  "trace_id" : "1-581cf771-a006649127e371903a2de979"
  “end_time” : 1.478293361449E9,
  “type” : “subsegment”,
  “parent_id” : “70de5b6f19ff9a0b”
}
```

For more information on the fields and values that you can include in segments and subsegments, see [AWS X\-Ray segment documents](xray-api-segmentdocuments.md)\.

**Topics**
+ [Generating trace IDs](#xray-api-traceids)
+ [Using PutTraceSegments](#xray-api-segments)
+ [Sending segment documents to the X\-Ray daemon](#xray-api-daemon)

## Generating trace IDs<a name="xray-api-traceids"></a>

To send data to X\-Ray, you need to generate a unique trace ID for each request\. Trace IDs must meet the following requirements\.

**Trace ID Format**

A `trace_id` consists of three numbers separated by hyphens\. For example, `1-58406520-a006649127e371903a2de979`\. This includes:
+ The version number, that is, `1`\.
+ The time of the original request, in Unix epoch time, in **8 hexadecimal digits**\.

  For example, 10:00AM December 1st, 2016 PST in epoch time is `1480615200` seconds, or `58406520` in hexadecimal digits\.
+ A 96\-bit identifier for the trace, globally unique, in **24 hexadecimal digits**\.

You can write a script to generate trace IDs for testing\. Here are two examples\.

**Python**

```
import time
import os
import binascii

START_TIME = time.time()
HEX=hex(int(START_TIME))[2:]
TRACE_ID="1-{}-{}".format(HEX, binascii.hexlify(os.urandom(12)))
```

**Bash**

```
START_TIME=$(date +%s)
HEX_TIME=$(printf '%x\n' $START_TIME)
GUID=$(dd if=/dev/random bs=12 count=1 2>/dev/null | od -An -tx1 | tr -d ' \t\n')
TRACE_ID="1-$HEX_TIME-$GUID"
```

See the Scorekeep sample application for scripts that create trace IDs and send segments to the X\-Ray daemon\.
+ Python – [https://github.com/awslabs/eb-java-scorekeep/blob/xray/bin/xray_start.py](https://github.com/awslabs/eb-java-scorekeep/blob/xray/bin/xray_start.py)
+ Bash – [https://github.com/awslabs/eb-java-scorekeep/blob/xray/bin/xray_start.sh](https://github.com/awslabs/eb-java-scorekeep/blob/xray/bin/xray_start.sh)

## Using PutTraceSegments<a name="xray-api-segments"></a>

You can upload segment documents with the [https://docs.aws.amazon.com/xray/latest/api/API_PutTraceSegments.html](https://docs.aws.amazon.com/xray/latest/api/API_PutTraceSegments.html) API\. The API has a single parameter, `TraceSegmentDocuments`, that takes a list of JSON segment documents\.

With the AWS CLI, use the `aws xray put-trace-segments` command to send segment documents directly to X\-Ray\.

```
$ DOC='{"trace_id": "1-5960082b-ab52431b496add878434aa25", "id": "6226467e3f845502", "start_time": 1498082657.37518, "end_time": 1498082695.4042, "name": "test.elasticbeanstalk.com"}'
$ aws xray put-trace-segments --trace-segment-documents "$DOC"
{
    "UnprocessedTraceSegments": []
}
```

**Note**  
Windows Command Processor and Windows PowerShell have different requirements for quoting and escaping quotes in JSON strings\. See [Quoting Strings](https://docs.aws.amazon.com/cli/latest/userguide/cli-using-param.html#quoting-strings) in the AWS CLI User Guide for details\.

The output lists any segments that failed processing\. For example, if the date in the trace ID is too far in the past, you see an error like the following\.

```
{
    "UnprocessedTraceSegments": [
        {
            "ErrorCode": "InvalidTraceId",
            "Message": "Invalid segment. ErrorCode: InvalidTraceId",
            "Id": "6226467e3f845502"
        }
    ]
}
```

You can pass multiple segment documents at the same time, separated by spaces\.

```
$ aws xray put-trace-segments --trace-segment-documents "$DOC1" "$DOC2"
```

## Sending segment documents to the X\-Ray daemon<a name="xray-api-daemon"></a>

Instead of sending segment documents to the X\-Ray API, you can send segments and subsegments to the X\-Ray daemon, which will buffer them and upload to the X\-Ray API in batches\. The X\-Ray SDK sends segment documents to the daemon to avoid making calls to AWS directly\.

**Note**  
See [Running the X\-Ray daemon locally](xray-daemon-local.md) for instructions on running the daemon\.

Send the segment in JSON over UDP port 2000, prepended by the daemon header, `{"format": "json", "version": 1}\n`

```
{"format": "json", "version": 1}\n{"trace_id": "1-5759e988-bd862e3fe1be46a994272793", "id": "defdfd9912dc5a56", "start_time": 1461096053.37518, "end_time": 1461096053.4042, "name": "test.elasticbeanstalk.com"}
```

On Linux, you can send segment documents to the daemon from a Bash terminal\. Save the header and segment document to a text file and pipe it to `/dev/udp` with `cat`\.

```
$ cat segment.txt > /dev/udp/127.0.0.1/2000
```

**Example segment\.txt**  

```
{"format": "json", "version": 1}
{"trace_id": "1-594aed87-ad72e26896b3f9d3a27054bb", "id": "6226467e3f845502", "start_time": 1498082657.37518, "end_time": 1498082695.4042, "name": "test.elasticbeanstalk.com"}
```

Check the [daemon log](xray-daemon.md#xray-daemon-logging) to verify that it sent the segment to X\-Ray\.

```
2017-07-07T01:57:24Z [Debug] processor: sending partial batch
2017-07-07T01:57:24Z [Debug] processor: segment batch size: 1. capacity: 50
2017-07-07T01:57:24Z [Info] Successfully sent batch of 1 segments (0.020 seconds)
```