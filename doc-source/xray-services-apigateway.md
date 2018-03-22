# Amazon API Gateway and AWS X\-Ray<a name="xray-services-apigateway"></a>

Amazon API Gateway gateways add a trace ID to incoming HTTP requests in a header named `X-Amzn-Trace-Id`\.

```
X-Amzn-Trace-Id: Root=1-5759e988-bd862e3fe1be46a994272793
```

**Trace ID Format**

A `trace_id` consists of three numbers separated by hyphens\. For example, `1-58406520-a006649127e371903a2de979`\. This includes:
+ The version number, that is, `1`\.
+ The time of the original request, in Unix epoch time, in **8 hexadecimal digits**\.

  For example, 10:00AM December 1st, 2016 PST in epoch time is `1480615200` seconds, or `58406520` in hexadecimal\.
+ A 96\-bit identifier for the trace, globally unique, in **24 hexadecimal digits**\.

API Gateway does not propagate X\-Ray trace ID and sampling headers, or send trace data to X\-Ray\. If your gateway is downstream of other services in your application, traces will terminate at the gateway and the request will continue with a different trace ID\. Gateways do not appear as a node on your service map\.