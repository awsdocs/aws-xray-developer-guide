# Instrumenting Incoming HTTP Requests with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-messagehandler"></a>

You can use the X\-Ray SDK to trace incoming HTTP requests that your application serves on an EC2 instance in Amazon EC2, AWS Elastic Beanstalk, or Amazon ECS\.

**Note**  
If your application runs on AWS Lambda, you can use the Lambda X\-Ray integration to trace incoming requests automatically\.

Use a `TracingMessageHandler` to instrument incoming HTTP requests\. When you add the X\-Ray message handler to your application, the X\-Ray SDK for \.NET creates a segment for each sampled request\. Any segments created by additional instrumentation become subsegments of the request\-level segment that provides information about the HTTP request and response, including timing, method, and disposition of the request\.

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


+ [Adding a Tracing Message Handler to your Application's HTTP Configuration](#xray-sdk-dotnet-messagehandler-webapiconfig)
+ [Adding a Tracing Message Handler to your Application Global Configuration](#xray-sdk-dotnet-messagehandler-globalasax)
+ [Configuring a Segment Naming Strategy](#xray-sdk-dotnet-messagehandler-naming)

## Adding a Tracing Message Handler to your Application's HTTP Configuration<a name="xray-sdk-dotnet-messagehandler-webapiconfig"></a>

To instrument requests served by your application, add a `TracingMessageHandler` to the `HttpConfiguration.MessageHandlers` collection in your web configuration\.

**Example WebApiConfig \- Message handler**  

```
using System.Web.Http;
using [Amazon\.XRay\.Recorder\.Handlers\.AspNet\.WebApi](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_AspNet_WebApi.htm);
using SampleEBWebApplication.Controllers;

namespace SampleEBWebApplication
{
  public static class WebApiConfig
  {
    public static void Register(HttpConfiguration config)
    {
      // Add the message handler to HttpConfiguration
      config.MessageHandlers.Add(new TracingMessageHandler("MyApp"));
      // Web API routes
      config.MapHttpAttributeRoutes();
      config.Routes.MapHttpRoute(
        name: "DefaultApi",
        routeTemplate: "api/{controller}/{id}",
        defaults: new { id = RouteParameter.Optional }
      );
    }
  }
}
```

## Adding a Tracing Message Handler to your Application Global Configuration<a name="xray-sdk-dotnet-messagehandler-globalasax"></a>

Alternatively, you can also add the tracing handler to a `global.asax` file\.

**Example global\.asax \- Message handler**  

```
using System.Web.Http;
using [Amazon\.XRay\.Recorder\.Handlers\.AspNet\.WebApi](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_AspNet_WebApi.htm);

namespace SampleEBWebApplication
{
  public class WebApiApplication : System.Web.HttpApplication
  {
    protected void Application_Start()
    {
      GlobalConfiguration.Configure(WebApiConfig.Register);
      GlobalConfiguration.Configuration.MessageHandlers.Add(new TracingMessageHandler("MyApp"));
    }
  }
}
```

## Configuring a Segment Naming Strategy<a name="xray-sdk-dotnet-messagehandler-naming"></a>

AWS X\-Ray uses a *service name* to identify your application and distinguish it from the other applications, databases, external APIs, and AWS resources that your application uses\. When the X\-Ray SDK generates segments for incoming requests, it records your application's service name in the segment's name field\.

The X\-Ray SDK can name segments after the hostname in the HTTP request header\. However, this header can be forged, which could result in unexpected nodes in your service map\. To prevent the SDK from naming segments incorrectly due to requests with forged host headers, you must specify a default name for incoming requests\.

If your application serves requests for multiple domains, you can configure the SDK to use a dynamic naming strategy to reflect this in segment names\. A dynamic naming strategy allows the SDK to use the hostname for requests that match an expected pattern, and apply the default name to requests that don't\.

For example, you might have a single application serving requests to three subdomains– `www.example.com`, `api.example.com`, and `static.example.com`\. You can use a dynamic naming strategy with the pattern `*.example.com` to identify segments for each subdomain with a different name, resulting in three service nodes on the service map\. If your application receives requests with a hostname that doesn't match the pattern, you will see a fourth node on the service map with a fallback name that you specify\.

To use the same name for all request segments, specify the name of your application when you initialize the servlet filter, as shown in the previous section\. This has the same effect as creating a [http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_FixedSegmentNamingStrategy.htm](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_FixedSegmentNamingStrategy.htm) and passing it to the [http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/M_Amazon_XRay_Recorder_Handlers_AspNet_WebApi_TracingMessageHandler__ctor.htm](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/M_Amazon_XRay_Recorder_Handlers_AspNet_WebApi_TracingMessageHandler__ctor.htm) constructor\.

```
config.MessageHandlers.Add(new TracingMessageHandler(new FixedSegmentNamingStrategy("MyApp"));
```

**Note**  
You can override the default service name that you define in code with the `AWS_XRAY_TRACING_NAME` environment variable\.

A dynamic naming strategy defines a pattern that hostnames should match, and a default name to use if the hostname in the HTTP request does not match the pattern\. To name segments dynamically, create a [http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_DynamicSegmentNamingStrategy.htm](http://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/T_Amazon_XRay_Recorder_Core_Strategies_DynamicSegmentNamingStrategy.htm) and pass it to the `TracingMessageHandler` constructor\.

```
config.MessageHandlers.Add(new TracingMessageHandler(new DynamicSegmentNamingStrategy("MyApp", "*.example.com"));
```