# Tracing incoming requests with the X\-Ray SDK for Java<a name="xray-sdk-java-filters"></a>

You can use the X\-Ray SDK to trace incoming HTTP requests that your application serves on an EC2 instance in Amazon EC2, AWS Elastic Beanstalk, or Amazon ECS\.

Use a `Filter` to instrument incoming HTTP requests\. When you add the X\-Ray servlet filter to your application, the X\-Ray SDK for Java creates a segment for each sampled request\. This segment includes timing, method, and disposition of the HTTP request\. Additional instrumentation creates subsegments on this segment\.

**Note**  
For AWS Lambda functions, Lambda creates a segment for each sampled request\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

Each segment has a name that identifies your application in the service map\. The segment can be named statically, or you can configure the SDK to name it dynamically based on the host header in the incoming request\. Dynamic naming lets you group traces based on the domain name in the request, and apply a default name if the name doesn't match an expected pattern \(for example, if the host header is forged\)\.

**Forwarded Requests**  
If a load balancer or other intermediary forwards a request to your application, X\-Ray takes the client IP from the `X-Forwarded-For` header in the request instead of from the source IP in the IP packet\. The client IP that is recorded for a forwarded request can be forged, so it should not be trusted\.

When a request is forwarded, the SDK sets an additional field in the segment to indicate this\. If the segment contains the field `x_forwarded_for` set to `true`, the client IP was taken from the `X-Forwarded-For` header in the HTTP request\.

The message handler creates a segment for each incoming request with an `http` block that contains the following information:
+ **HTTP method** – GET, POST, PUT, DELETE, etc\.
+ **Client address** – The IP address of the client that sent the request\.
+ **Response code** – The HTTP response code for the completed request\.
+ **Timing** – The start time \(when the request was received\) and end time \(when the response was sent\)\.
+ **User agent** — The `user-agent` from the request\.
+ **Content length** — The `content-length` from the response\.

**Topics**
+ [Adding a tracing filter to your application \(Tomcat\)](#xray-sdk-java-filters-tomcat)
+ [Adding a tracing filter to your application \(spring\)](#xray-sdk-java-filters-spring)
+ [Configuring a segment naming strategy](#xray-sdk-java-filters-naming)

## Adding a tracing filter to your application \(Tomcat\)<a name="xray-sdk-java-filters-tomcat"></a>

For Tomcat, add a `<filter>` to your project's `web.xml` file\. Use the `fixedName` parameter to specify a [service name](#xray-sdk-java-filters-naming) to apply to segments created for incoming requests\.

**Example WEB\-INF/web\.xml \- Tomcat**  

```
<filter>
  <filter-name>AWSXRayServletFilter</filter-name>
  <filter-class>com.amazonaws.xray.javax.servlet.AWSXRayServletFilter</filter-class>
  <init-param>
    <param-name>fixedName</param-name>
    <param-value>MyApp</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>AWSXRayServletFilter</filter-name>
  <url-pattern>*</url-pattern>
</filter-mapping>
```

## Adding a tracing filter to your application \(spring\)<a name="xray-sdk-java-filters-spring"></a>

For Spring, add a `Filter` to your `WebConfig` class\. Pass the segment name to the [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html) constructor as a string\.

**Example src/main/java/myapp/WebConfig\.java \- spring**  

```
package myapp;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import javax.servlet.Filter;
import [com\.amazonaws\.xray\.javax\.servlet\.AWSXRayServletFilter](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html);

@Configuration
public class WebConfig {

  @Bean
  public Filter TracingFilter() {
    return new AWSXRayServletFilter("Scorekeep");
  }
}
```

## Configuring a segment naming strategy<a name="xray-sdk-java-filters-naming"></a>

AWS X\-Ray uses a *service name* to identify your application and distinguish it from the other applications, databases, external APIs, and AWS resources that your application uses\. When the X\-Ray SDK generates segments for incoming requests, it records your application's service name in the segment's [name field](xray-api-segmentdocuments.md#api-segmentdocuments-fields)\.

The X\-Ray SDK can name segments after the hostname in the HTTP request header\. However, this header can be forged, which could result in unexpected nodes in your service map\. To prevent the SDK from naming segments incorrectly due to requests with forged host headers, you must specify a default name for incoming requests\.

If your application serves requests for multiple domains, you can configure the SDK to use a dynamic naming strategy to reflect this in segment names\. A dynamic naming strategy allows the SDK to use the hostname for requests that match an expected pattern, and apply the default name to requests that don't\.

For example, you might have a single application serving requests to three subdomains– `www.example.com`, `api.example.com`, and `static.example.com`\. You can use a dynamic naming strategy with the pattern `*.example.com` to identify segments for each subdomain with a different name, resulting in three service nodes on the service map\. If your application receives requests with a hostname that doesn't match the pattern, you will see a fourth node on the service map with a fallback name that you specify\.

To use the same name for all request segments, specify the name of your application when you initialize the servlet filter, as shown in [the previous section](#xray-sdk-java-filters-tomcat)\. This has the same effect as creating a [FixedSegmentNamingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/FixedSegmentNamingStrategy.html) and passing it to [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html) constructor\.

**Note**  
You can override the default service name that you define in code with the `AWS_XRAY_TRACING_NAME` [environment variable](xray-sdk-java-configuration.md#xray-sdk-java-configuration-envvars)\.

A dynamic naming strategy defines a pattern that hostnames should match, and a default name to use if the hostname in the HTTP request does not match the pattern\. To name segments dynamically in Tomcat, use the `dynamicNamingRecognizedHosts` and `dynamicNamingFallbackName` to define the pattern and default name, respectively\.

**Example WEB\-INF/web\.xml \- servlet filter with dynamic naming**  

```
<filter>
  <filter-name>AWSXRayServletFilter</filter-name>
  <filter-class>com.amazonaws.xray.javax.servlet.AWSXRayServletFilter</filter-class>
  <init-param>
    <param-name>dynamicNamingRecognizedHosts</param-name>
    <param-value>*.example.com</param-value>
  </init-param>
  <init-param>
    <param-name>dynamicNamingFallbackName</param-name>
    <param-value>MyApp</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>AWSXRayServletFilter</filter-name>
  <url-pattern>*</url-pattern>
</filter-mapping>
```

For Spring, create a [DynamicSegmentNamingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/DynamicSegmentNamingStrategy.html) and pass it to the `AWSXRayServletFilter` constructor\.

**Example src/main/java/myapp/WebConfig\.java \- servlet filter with dynamic naming**  

```
package myapp;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import javax.servlet.Filter;
import [com\.amazonaws\.xray\.javax\.servlet\.AWSXRayServletFilter](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html);
import [com\.amazonaws\.xray\.strategy\.DynamicSegmentNamingStrategy](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/strategy/DynamicSegmentNamingStrategy.html);

@Configuration
public class WebConfig {

  @Bean
  public Filter TracingFilter() {
    return new AWSXRayServletFilter(new DynamicSegmentNamingStrategy("MyApp", "*.example.com"));
  }
}
```