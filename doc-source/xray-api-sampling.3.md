# Using sampling rules with the X\-Ray API<a name="xray-api-sampling"></a>

The AWS X\-Ray SDK uses the X\-Ray API to get sampling rules, report sampling results, and get quotas\. You can use these APIs to get a better understanding of how sampling rules work, or to implement sampling in a language that the X\-Ray SDK doesn't support\.

Start by getting all sampling rules with [https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html](https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html)\.

```
$ aws xray get-sampling-rules
{
    "SamplingRuleRecords": [
        {
            "SamplingRule": {
                "RuleName": "Default",
                "RuleARN": "arn:aws:xray:us-east-1::sampling-rule/Default",
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
            "ModifiedAt": 1530558121.0
        },
        {
            "SamplingRule": {
                "RuleName": "base-scorekeep",
                "RuleARN": "arn:aws:xray:us-east-1::sampling-rule/base-scorekeep",
                "ResourceARN": "*",
                "Priority": 9000,
                "FixedRate": 0.1,
                "ReservoirSize": 2,
                "ServiceName": "Scorekeep",
                "ServiceType": "*",
                "Host": "*",
                "HTTPMethod": "*",
                "URLPath": "*",
                "Version": 1,
                "Attributes": {}
            },
            "CreatedAt": 1530573954.0,
            "ModifiedAt": 1530920505.0
        },
        {
            "SamplingRule": {
                "RuleName": "polling-scorekeep",
                "RuleARN": "arn:aws:xray:us-east-1::sampling-rule/polling-scorekeep",
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
            "CreatedAt": 1530918163.0,
            "ModifiedAt": 1530918163.0
        }
    ]
}
```

The output includes the default rule and custom rules\. See [Sampling rules](xray-api-configuration.md#xray-api-configuration-sampling) if you haven't yet created sampling rules\.

Evaluate rules against incoming requests in ascending order of priority\. When a rule matches, use the fixed rate and reservoir size to make a sampling decision\. Record sampled requests and ignore \(for tracing purposes\) unsampled requests\. Stop evaluating rules when a sampling decision is made\.

A rules reservoir size is the target number of traces to record per second before applying the fixed rate\. The reservoir applies across all services cumulatively, so you can't use it directly\. However, if it is non\-zero, you can borrow one trace per second from the reservoir until X\-Ray assigns a quota\. Before receiving a quota, record the first request each second, and apply the fixed rate to additional requests\. The fixed rate is a decimal between 0 and 1\.00 \(100%\)\.

The following example shows a call to [https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingTargets.html](https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingTargets.html) with details about sampling decisions made over the last 10 seconds\.

```
$ aws xray get-sampling-targets --sampling-statistics-documents '[
    {
        "RuleName": "base-scorekeep",
        "ClientID": "ABCDEF1234567890ABCDEF10",
        "Timestamp": "2018-07-07T00:20:06,
        "RequestCount": 110,
        "SampledCount": 20,
        "BorrowCount": 10
    },
    {
        "RuleName": "polling-scorekeep",
        "ClientID": "ABCDEF1234567890ABCDEF10",
        "Timestamp": "2018-07-07T00:20:06",
        "RequestCount": 10500,
        "SampledCount": 31,
        "BorrowCount": 0
    }
]'
{
    "SamplingTargetDocuments": [
        {
            "RuleName": "base-scorekeep",
            "FixedRate": 0.1,
            "ReservoirQuota": 2,
            "ReservoirQuotaTTL": 1530923107.0,
            "Interval": 10
        },
        {
            "RuleName": "polling-scorekeep",
            "FixedRate": 0.003,
            "ReservoirQuota": 0,
            "ReservoirQuotaTTL": 1530923107.0,
            "Interval": 10
        }
    ],
    "LastRuleModification": 1530920505.0,
    "UnprocessedStatistics": []
}
```

The response from X\-Ray includes a quota to use instead of borrowing from the reservoir\. In this example, the service borrowed 10 traces from the reservoir over 10 seconds, and applied the fixed rate of 10 percent to the other 100 requests, resulting in a total of 20 sampled requests\. The quota is good for five minutes \(indicated by the time to live\) or until a new quota is assigned\. X\-Ray may also assign a longer reporting interval than the default, although it didn't here\.

**Note**  
The response from X\-Ray might not include a quota the first time you call it\. Continue borrowing from the reservoir until you are assigned a quota\.

The other two fields in the response might indicate issues with the input\. Check `LastRuleModification` against the last time you called [https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html](https://docs.aws.amazon.com/xray/latest/api/API_GetSamplingRules.html)\. If it's newer, get a new copy of the rules\. `UnprocessedStatistics` can include errors that indicate that a rule has been deleted, that the statistics document in the input was too old, or permissions errors\.