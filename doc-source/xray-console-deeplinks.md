# Deep linking<a name="xray-console-deeplinks"></a>

You can use routes and queries to deep link into specific traces, or filtered views of traces and the service map\.

**Console pages**
+ Welcome page – [xray/home\#/welcome](https://console.aws.amazon.com/xray/home#/welcome)
+ Getting started – [xray/home\#/getting\-started](https://console.aws.amazon.com/xray/home#/getting-started)
+ Service map – [xray/home\#/service\-map](https://console.aws.amazon.com/xray/home#/service-map)
+ Traces – [xray/home\#/traces](https://console.aws.amazon.com/xray/home#/traces)

## Traces<a name="xray-console-deeplinks-traces"></a>

You can generate links for timeline, raw, and map views of individual traces\.

**Trace timeline** – `xray/home#/traces/trace-id`

**Raw trace data** – `xray/home#/traces/trace-id/raw`

**Example – raw trace data**  

```
https://console.aws.amazon.com/xray/home#/traces/1-57f5498f-d91047849216d0f2ea3b6442/raw
```

## Filter expressions<a name="xray-console-deeplinks-filters"></a>

Link to a filtered list of traces\.

**Filtered traces view** – `xray/home#/traces?filter=filter-expression`

**Example – filter expression**  

```
https://console.aws.amazon.com/xray/home#/traces?filter=service("api.amazon.com") { fault = true OR responsetime > 2.5 } AND annotation.foo = "bar"
```

**Example – filter expression \(URL encoded\)**  

```
https://console.aws.amazon.com/xray/home#/traces?filter=service(%22api.amazon.com%22)%20%7B%20fault%20%3D%20true%20OR%20responsetime%20%3E%202.5%20%7D%20AND%20annotation.foo%20%3D%20%22bar%22
```

For more information about filter expressions, see [Using filter expressions to search for traces in the console](xray-console-filters.md)\.

## Time range<a name="xray-console-deeplinks-time"></a>

Specify a length of time or start and end time in ISO8601 format\. Time ranges are in UTC and can be up to 6 hours long\.

**Length of time** – `xray/home#/page?timeRange=range-in-minutes` 

**Example – service map for the last hour**  

```
https://console.aws.amazon.com/xray/home#/service-map?timeRange=PT1H
```

**Start and end time** – `xray/home#/page?timeRange=start~end` 

**Example – time range accurate to seconds**  

```
https://console.aws.amazon.com/xray/home#/traces?timeRange=2018-9-01T16:00:00~2018-9-01T22:00:00
```

**Example – time range accurate to minutes**  

```
https://console.aws.amazon.com/xray/home#/traces?timeRange=2018-9-01T16:00~2018-9-01T22:00
```

## Region<a name="xray-console-deeplinks-region"></a>

Specify an AWS Region to link to pages in that Region\. If you don't specify a Region, the console redirects you to the last visited Region\.

**Region** – `xray/home?region=region#/page` 

**Example – service map in US West \(Oregon\) \(us\-west\-2\)**  

```
https://console.aws.amazon.com/xray/home?region=us-west-2#/service-map
```

When you include a Region with other query parameters, the Region query goes before the hash, and the X\-Ray\-specific queries go after the page name\.

**Example – service map for the last hour in US West \(Oregon\) \(us\-west\-2\)**  

```
https://console.aws.amazon.com/xray/home?region=us-west-2#/service-map?timeRange=PT1H
```

## Combined<a name="xray-console-deeplinks-combined"></a>

**Example – recent traces with a duration filter**  

```
https://console.aws.amazon.com/xray/home#/traces?timeRange=PT15M&filter=duration%20%3E%3D%205%20AND%20duration%20%3C%3D%208
```

**Output**
+ Page – Traces
+ Time Range – Last 15 minutes
+ Filter – duration >= 5 AND duration <= 8