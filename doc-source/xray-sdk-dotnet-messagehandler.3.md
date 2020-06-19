# Instrumenting incoming HTTP requests with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-messagehandler"></a>

You can use the X\-Ray SDK to trace incoming HTTP requests that your application serves on an EC2 instance in Amazon EC2, AWS Elastic Beanstalk, or Amazon ECS\.

Use a message handler to instrument incoming HTTP requests\. When you add the X\-Ray message handler to your application, the X\-Ray SDK for \.NET creates a segment for each sampled request\. This segment includes timing, method, and disposition of the HTTP request\. Additional instrumentation creates subsegments on this segment\.

**Note**  
For AWS Lambda functions, Lambda creates a segment for each sampled request\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

Each segment has a name that identifies your application in the service map\. The segment can be named statically, or you can configure the SDK to name it dynamically based on the host header in the incoming request\. Dynamic naming lets you group traces based on the domain name in the request, and apply a default name if the name doesn't match an expected pattern \(for example, if the host header is forged\)\.

**Forwarded Requests**  
If a load balancer or other intermediary forwards a request to your application, X\-Ray takes the client IP from the `X-Forwarded-For` header in the request instead of from the source IP in the IP packet\. The client IP that is recorded for a forwarded request can be forged, so it should not be trusted\.

The message handler creates a segment for each incoming request with an `http` block that contains the following information:
+ **HTTP method** – GET, POST, PUT, DELETE, etc\.
+ **Client address** – The IP address of the client that sent the request\.
+ **Response code** – The HTTP response code for the completed request\.
+ **Timing** – The start time \(when the request was received\) and end time \(when the response was sent\)\.
+ **User agent** — The `user-agent` from the request\.
+ **Content length** — The `content-length` from the response\.

**Topics**
+ [Instrumenting incoming requests \(\.NET\)](#xray-sdk-dotnet-messagehandler-globalasax)
+ [Instrumenting incoming requests \(\.NET Core\)](#xray-sdk-dotnet-messagehandler-startupcs)
+ [Configuring a segment naming strategy](#xray-sdk-dotnet-messagehandler-naming)

## Instrumenting incoming requests \(\.NET\)<a name="xray-sdk-dotnet-messagehandler-globalasax"></a>

To instrument requests served by your application, call `RegisterXRay` in the `Init` method of your `global.asax` file\.

**Example global\.asax \- message handler**  

```
using System.Web.Http;
using [Amazon\.XRay\.Recorder\.Handlers\.AspNet](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_AspNet.htm);

namespace SampleEBWebApplication
{
  public class MvcApplication : System.Web.HttpApplication
  {
    public override void Init()
    {
      base.Init();
      AWSXRayASPNET.RegisterXRay(this, "MyApp");
    }
  }
}
```

## Instrumenting incoming requests \(\.NET Core\)<a name="xray-sdk-dotnet-messagehandler-startupcs"></a>

To instrument requests served by your application, call the `UseExceptionHandler`, `UseXRay`, and `UseStaticFiles` methods in the `Configure` method of your `Startup` class\.

**Example Startup\.cs**  

```
using Microsoft.AspNetCore.Builder;

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
  {
    app.UseExceptionHandler("/Error");
    app.UseXRay("MyApp");
    app.UseStaticFiles();
    app.UseMVC();
  }
```

Always call `UseXRay` after `UseExceptionHandler` to record exceptions\. If you use other middleware, enable it after you call `UseXRay`\.

The `UseXRay` method can also take a [configuration object](xray-sdk-dotnet-configuration.md) as a second argument\.

```
app.UseXRay("MyApp", configuration);
```

## Configuring a segment naming strategy<a name="xray-sdk-dotnet-messagehandler-naming"></a>

AWS X\-Ray uses a *service name* to identify your application and distinguish it from the other applications, databases, external APIs, and AWS resources that your application uses\. When the X\-Ray SDK generates segments for incoming requests, it records your application's service name in the segment's [name field](xray-api-segmentdocuments.md#api-segmentdocuments-fields)\.

The X\-Ray SDK can name segments after the hostname in the HTTP request header\. However, this header can be forged, which could result in unexpected nodes in your service map\. To prevent the SDK from naming segments incorrectly due to requests with forged host headers, you must specify a default name for incoming requests\.

If your application serves requests for multiple domains, you can configure the SDK to use a dynamic naming strategy to reflect this in segment names\. A dynamic naming strategy allows the SDK to use the hostname for requests that match an expected pattern, and apply the default name to requests that don't\.

For example, you might have a single application serving requests to three subdomains– `www.example.com`, `api.example.com`, and `static.example.com`\. You can use a dynamic naming strategy with the pattern `*.example.com` to identify segments for each subdomain with a different name, resulting in three service nodes on the service map\. If your application receives requests with a hostname that doesn't match the pattern, you will see a fourth node on the service map with a fallback name that you specify\.

To use the same name for all request segments, specify the name of your application when you initialize the message handler, as shown in [the previous section](#xray-sdk-dotnet-messagehandler-globalasax)\. This has the same effect as creating a [https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_FixedSegmentNamingStrategy.htm](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_FixedSegmentNamingStrategy.htm) and passing it to the `RegisterXRay` method\.

```
AWSXRayASPNET.RegisterXRay(this, new FixedSegmentNamingStrategy("MyApp"));
```

**Note**  
You can override the default service name that you define in code with the `AWS_XRAY_TRACING_NAME` [environment variable](xray-sdk-dotnet-configuration.md#xray-sdk-dotnet-configuration-envvars)\.

A dynamic naming strategy defines a pattern that hostnames should match, and a default name to use if the hostname in the HTTP request does not match the pattern\. To name segments dynamically, create a [https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_DynamicSegmentNamingStrategy.htm](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_DynamicSegmentNamingStrategy.htm) and pass it to the `RegisterXRay` method\.

```
AWSXRayASPNET.RegisterXRay(this, new DynamicSegmentNamingStrategy("MyApp", "*.example.com"));
```