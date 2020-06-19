# Elastic Load Balancing and AWS X\-Ray<a name="xray-services-elb"></a>

Elastic Load Balancing application load balancers add a trace ID to incoming HTTP requests in a header named `X-Amzn-Trace-Id`\.

```
X-Amzn-Trace-Id: Root=1-5759e988-bd862e3fe1be46a994272793
```

**Trace ID Format**

A `trace_id` consists of three numbers separated by hyphens\. For example, `1-58406520-a006649127e371903a2de979`\. This includes:
+ The version number, that is, `1`\.
+ The time of the original request, in Unix epoch time, in **8 hexadecimal digits**\.

  For example, 10:00AM December 1st, 2016 PST in epoch time is `1480615200` seconds, or `58406520` in hexadecimal digits\.
+ A 96\-bit identifier for the trace, globally unique, in **24 hexadecimal digits**\.

Load balancers do not send data to X\-Ray, and do not appear as a node on your service map\.

For more information, see [Request Tracing for Your Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-request-tracing.html) in the Elastic Load Balancing Developer Guide\.