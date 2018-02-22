# Deep Linking<a name="xray-console-deeplinks"></a>

You can use routes and queries to deep link into specific traces, or filtered views of traces and the service map\.

**Console Pages**

+ Welcome Page – [xray/home\#/welcome](https://console.aws.amazon.com/xray/home#/welcome)

+ Getting Started – [xray/home\#/getting\-started](https://console.aws.amazon.com/xray/home#/getting-started)

+ Service Map – [xray/home\#/service\-map](https://console.aws.amazon.com/xray/home#/service-map)

+ Traces – [xray/home\#/traces](https://console.aws.amazon.com/xray/home#/traces)

## Traces<a name="xray-console-deeplinks-traces"></a>

You can generate links for timeline, raw, and map views of individual traces\.

**Trace timeline** – `xray/home#/traces/trace-id`

**Raw trace data** – `xray/home#/traces/trace-id/raw`

**Example Raw trace data**  

```
https://console.aws.amazon.com/xray/home#/traces/1-57f5498f-d91047849216d0f2ea3b6442/raw
```

## Filter Expressions<a name="xray-console-deeplinks-filters"></a>

Link to a filtered list of traces\.

**Filtered traces view** – `xray/home#/traces?filter=filter-expression`

**Example Filter expression**  

```
https://console.aws.amazon.com/xray/home#/traces?filter=service("api.amazon.com") { fault = true OR responsetime > 2.5 } AND annotation.foo = "bar"
```

**Example Filter expression \(URL encoded\)**  

```
https://console.aws.amazon.com/xray/home#/traces?filter=service(%22api.amazon.com%22)%20%7B%20fault%20%3D%20true%20OR%20responsetime%20%3E%202.5%20%7D%20AND%20annotation.foo%20%3D%20%22bar%22
```

For more information on filter expressions, see [Searching for Traces in the AWS X\-Ray Console with Filter Expressions](xray-console-filters.md)\.

## Time Range<a name="xray-console-deeplinks-time"></a>

Specify a length of time or start and end time in ISO8601 format\. Time ranges are in UTC and can be up to 6 hours long\.

**Length of time** – `xray/home#/page?timeRange=range-in-minutes` 

**Example Service map for the last hour**  

```
https://console.aws.amazon.com/xray/home#/service-map?timeRange=PT1H
```

**Start and end time** – `xray/home#/page?timeRange=start~end` 

**Example Time range accurate to seconds**  

```
https://console.aws.amazon.com/xray/home#/traces?timeRange=2018-2-01T16:00:00~2018-2-01T22:00:00
```

**Example Time range accurate to minutes**  

```
https://console.aws.amazon.com/xray/home#/traces?timeRange=2018-2-01T16:00~2018-2-01T22:00
```

## Region<a name="xray-console-deeplinks-region"></a>

Specify a region to link to pages in that region\. If you don't specify a region, the console redirects you to the last visited region\.

**Region** – `xray/home?region=region#/page` 

**Example Service map in Oregon \(us\-west\-2\)**  

```
https://console.aws.amazon.com/xray/home?region=us-west-2#/service-map
```

When you include a region with other query parameters, the region query goes before the hash, and the X\-Ray\-specific queries go after the page name\.

**Example Service map for the last hour in Oregon \(us\-west\-2\)**  

```
https://console.aws.amazon.com/xray/home?region=us-west-2#/service-map?timeRange=PT1H
```

## Combined<a name="xray-console-deeplinks-combined"></a>

**Example Recent traces with duration filter**  

```
https://console.aws.amazon.com/xray/home#/traces?timeRange=PT15M&filter=duration%20%3E%3D%205%20AND%20duration%20%3C%3D%208
```

**Output**

+ Page – Traces

+ Time Range – Last 15 minutes

+ Filter – duration >= 5 AND duration <= 8