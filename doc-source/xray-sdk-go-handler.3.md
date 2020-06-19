# Instrumenting incoming HTTP requests with the X\-Ray SDK for Go<a name="xray-sdk-go-handler"></a>

You can use the X\-Ray SDK to trace incoming HTTP requests that your application serves on an EC2 instance in Amazon EC2, AWS Elastic Beanstalk, or Amazon ECS\.

Use `xray.Handler` to instrument incoming HTTP requests\. The X\-Ray SDK for Go implements the standard Go library `http.Handler` interface in the `xay.Handler` class to intercept web requests\. The `xay.Handler` class wraps the provided `http.Handler` with `xray.Capture` using the request's context, parsing the incoming headers, adding response headers if needed, and sets HTTP\-specific trace fields\.

When you use this class to handle HTTP requests and responses, the X\-Ray SDK for Go creates a segment for each sampled request\. This segment includes timing, method, and disposition of the HTTP request\. Additional instrumentation creates subsegments on this segment\.

**Note**  
For AWS Lambda functions, Lambda creates a segment for each sampled request\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

The following example intercepts requests on port 8000 and returns "Hello\!" as a response\. It creates the segment `myApp` and instruments calls through any application\.

**Example main\.go**  

```
func main() {
  http.Handle("/", xray.Handler(xray.NewFixedSegmentNamer("MyApp"), http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello!"))
  })))

  http.ListenAndServe(":8000", nil)
}
```

Each segment has a name that identifies your application in the service map\. The segment can be named statically, or you can configure the SDK to name it dynamically based on the host header in the incoming request\. Dynamic naming lets you group traces based on the domain name in the request, and apply a default name if the name doesn't match an expected pattern \(for example, if the host header is forged\)\.

**Forwarded Requests**  
If a load balancer or other intermediary forwards a request to your application, X\-Ray takes the client IP from the `X-Forwarded-For` header in the request instead of from the source IP in the IP packet\. The client IP that is recorded for a forwarded request can be forged, so it should not be trusted\.

When a request is forwarded, the SDK sets an additional field in the segment to indicate this\. If the segment contains the field `x_forwarded_for` set to `true`, the client IP was taken from the `X-Forwarded-For` header in the HTTP request\.

The handler creates a segment for each incoming request with an `http` block that contains the following information:
+ **HTTP method** – GET, POST, PUT, DELETE, etc\.
+ **Client address** – The IP address of the client that sent the request\.
+ **Response code** – The HTTP response code for the completed request\.
+ **Timing** – The start time \(when the request was received\) and end time \(when the response was sent\)\.
+ **User agent** — The `user-agent` from the request\.
+ **Content length** — The `content-length` from the response\.

## Configuring a segment naming strategy<a name="xray-sdk-go-segments-naming"></a>

AWS X\-Ray uses a *service name* to identify your application and distinguish it from the other applications, databases, external APIs, and AWS resources that your application uses\. When the X\-Ray SDK generates segments for incoming requests, it records your application's service name in the segment's [name field](xray-api-segmentdocuments.md#api-segmentdocuments-fields)\.

The X\-Ray SDK can name segments after the hostname in the HTTP request header\. However, this header can be forged, which could result in unexpected nodes in your service map\. To prevent the SDK from naming segments incorrectly due to requests with forged host headers, you must specify a default name for incoming requests\.

If your application serves requests for multiple domains, you can configure the SDK to use a dynamic naming strategy to reflect this in segment names\. A dynamic naming strategy allows the SDK to use the hostname for requests that match an expected pattern, and apply the default name to requests that don't\.

For example, you might have a single application serving requests to three subdomains– `www.example.com`, `api.example.com`, and `static.example.com`\. You can use a dynamic naming strategy with the pattern `*.example.com` to identify segments for each subdomain with a different name, resulting in three service nodes on the service map\. If your application receives requests with a hostname that doesn't match the pattern, you will see a fourth node on the service map with a fallback name that you specify\.

To use the same name for all request segments, specify the name of your application when you create the handler, as shown in the previous section\.

**Note**  
You can override the default service name that you define in code with the `AWS_XRAY_TRACING_NAME` [environment variable](xray-sdk-go-configuration.md#xray-sdk-go-configuration-envvars)\.

A dynamic naming strategy defines a pattern that hostnames should match, and a default name to use if the hostname in the HTTP request doesn't match the pattern\. To name segments dynamically, use `NewDynamicSegmentNamer` to configure the default name and pattern to match\.

**Example main\.go**  
If the hostname in the request matches the pattern `*.example.com`, use the hostname\. Otherwise, use `MyApp`\.  

```
func main() {
  http.Handle("/", xray.Handler(xray.NewDynamicSegmentNamer("MyApp", "*.example.com"), http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello!"))
  })))

  http.ListenAndServe(":8000", nil)
}
```