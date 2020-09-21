# Tracing incoming requests with the X\-Ray SDK for Python middleware<a name="xray-sdk-python-middleware"></a>

When you add the middleware to your application and configure a segment name, the X\-Ray SDK for Python creates a segment for each sampled request\. This segment includes timing, method, and disposition of the HTTP request\. Additional instrumentation creates subsegments on this segment\.

The X\-Ray SDK for Python supports the following middleware to instrument incoming HTTP requests: 
+ Django
+ Flask
+ Bottle

**Note**  
For AWS Lambda functions, Lambda creates a segment for each sampled request\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

See [Worker](scorekeep-lambda.md#scorekeep-lambda-worker) for a example Python function instrumented in Lambda\.

For scripts or Python applications on other frameworks, you can [create segments manually](#xray-sdk-python-middleware-manual)\.

Each segment has a name that identifies your application in the service map\. The segment can be named statically, or you can configure the SDK to name it dynamically based on the host header in the incoming request\. Dynamic naming lets you group traces based on the domain name in the request, and apply a default name if the name doesn't match an expected pattern \(for example, if the host header is forged\)\.

**Forwarded Requests**  
If a load balancer or other intermediary forwards a request to your application, X\-Ray takes the client IP from the `X-Forwarded-For` header in the request instead of from the source IP in the IP packet\. The client IP that is recorded for a forwarded request can be forged, so it should not be trusted\.

When a request is forwarded, the SDK sets an additional field in the segment to indicate this\. If the segment contains the field `x_forwarded_for` set to `true`, the client IP was taken from the `X-Forwarded-For` header in the HTTP request\.

The middleware creates a segment for each incoming request with an `http` block that contains the following information:
+ **HTTP method** – GET, POST, PUT, DELETE, etc\.
+ **Client address** – The IP address of the client that sent the request\.
+ **Response code** – The HTTP response code for the completed request\.
+ **Timing** – The start time \(when the request was received\) and end time \(when the response was sent\)\.
+ **User agent** — The `user-agent` from the request\.
+ **Content length** — The `content-length` from the response\.

**Topics**
+ [Adding the middleware to your application \(Django\)](#xray-sdk-python-adding-middleware-django)
+ [Adding the middleware to your application \(flask\)](#xray-sdk-python-adding-middleware-flask)
+ [Adding the middleware to your application \(Bottle\)](#xray-sdk-python-adding-middleware-bottle)
+ [Instrumenting Python code manually](#xray-sdk-python-middleware-manual)
+ [Configuring a segment naming strategy](#xray-sdk-python-middleware-naming)

## Adding the middleware to your application \(Django\)<a name="xray-sdk-python-adding-middleware-django"></a>

Add the middleware to the `MIDDLEWARE` list in your `settings.py` file\. The X\-Ray middleware should be the first line in your `settings.py` file to ensure that requests that fail in other middleware are recorded\.

**Example settings\.py \- X\-Ray SDK for Python middleware**  

```
MIDDLEWARE = [
    'aws_xray_sdk.ext.django.middleware.XRayMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware'
]
```

Configure a segment name in your [`settings.py` file](xray-sdk-python-configuration.md#xray-sdk-python-middleware-configuration-django)\.

**Example settings\.py – Segment name**  

```
XRAY_RECORDER = {,
    'AWS_XRAY_TRACING_NAME': 'My application',
    'PLUGINS': ('EC2Plugin'),
}
```

This tells the X\-Ray recorder to trace requests served by your Django application with the default sampling rate\. You can [configure the recorder your Django settings file](xray-sdk-python-configuration.md#xray-sdk-python-middleware-configuration-django) to apply custom sampling rules or change other settings\.

## Adding the middleware to your application \(flask\)<a name="xray-sdk-python-adding-middleware-flask"></a>

To instrument your Flask application, first configure a segment name on the `xray_recorder`\. Then, use the `XRayMiddleware` function to patch your Flask application in code\.

**Example app\.py**  

```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

app = Flask(__name__)

xray_recorder.configure(service='My application')
XRayMiddleware(app, xray_recorder)
```

This tells the X\-Ray recorder to trace requests served by your Flask application with the default sampling rate\. You can [configure the recorder in code](xray-sdk-python-configuration.md#xray-sdk-python-middleware-configuration-code) to apply custom sampling rules or change other settings\.

## Adding the middleware to your application \(Bottle\)<a name="xray-sdk-python-adding-middleware-bottle"></a>

To instrument your Bottle application, first configure a segment name on the `xray_recorder`\. Then, use the `XRayMiddleware` function to patch your Bottle application in code\.

**Example app\.py**  

```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.bottle.middleware import XRayMiddleware
 
app = Bottle()
 
xray_recorder.configure(service='fallback_name', dynamic_naming='My application')
app.install(XRayMiddleware(xray_recorder))
```

This tells the X\-Ray recorder to trace requests served by your Bottle application with the default sampling rate\. You can [configure the recorder in code](xray-sdk-python-configuration.md#xray-sdk-python-middleware-configuration-code) to apply custom sampling rules or change other settings\.

## Instrumenting Python code manually<a name="xray-sdk-python-middleware-manual"></a>

If you don't use Django or Flask, you can create segments manually\. You can create a segment for each incoming request, or create segments around patched HTTP or AWS SDK clients to provide context for the recorder to add subsegments\.

**Example main\.py – Manual instrumentation**  

```
from aws_xray_sdk.core import xray_recorder

# Start a segment
segment = xray_recorder.begin_segment('segment_name')
# Start a subsegment
subsegment = xray_recorder.begin_subsegment('subsegment_name')

# Add metadata and annotations
segment.put_metadata('key', dict, 'namespace')
subsegment.put_annotation('key', 'value')

# Close the subsegment and segment
xray_recorder.end_subsegment()
xray_recorder.end_segment()
```

## Configuring a segment naming strategy<a name="xray-sdk-python-middleware-naming"></a>

AWS X\-Ray uses a *service name* to identify your application and distinguish it from the other applications, databases, external APIs, and AWS resources that your application uses\. When the X\-Ray SDK generates segments for incoming requests, it records your application's service name in the segment's [name field](xray-api-segmentdocuments.md#api-segmentdocuments-fields)\.

The X\-Ray SDK can name segments after the hostname in the HTTP request header\. However, this header can be forged, which could result in unexpected nodes in your service map\. To prevent the SDK from naming segments incorrectly due to requests with forged host headers, you must specify a default name for incoming requests\.

If your application serves requests for multiple domains, you can configure the SDK to use a dynamic naming strategy to reflect this in segment names\. A dynamic naming strategy allows the SDK to use the hostname for requests that match an expected pattern, and apply the default name to requests that don't\.

For example, you might have a single application serving requests to three subdomains– `www.example.com`, `api.example.com`, and `static.example.com`\. You can use a dynamic naming strategy with the pattern `*.example.com` to identify segments for each subdomain with a different name, resulting in three service nodes on the service map\. If your application receives requests with a hostname that doesn't match the pattern, you will see a fourth node on the service map with a fallback name that you specify\.

To use the same name for all request segments, specify the name of your application when you configure the recorder, as shown in the [previous sections](#xray-sdk-python-adding-middleware-django)\.

A dynamic naming strategy defines a pattern that hostnames should match, and a default name to use if the hostname in the HTTP request doesn't match the pattern\. To name segments dynamically in Django, add the `DYNAMIC_NAMING` setting to your [settings\.py](xray-sdk-python-configuration.md#xray-sdk-python-middleware-configuration-django) file\.

**Example settings\.py – Dynamic naming**  

```
XRAY_RECORDER = {
    'AUTO_INSTRUMENT': True,
    'AWS_XRAY_TRACING_NAME': 'My application',
    'DYNAMIC_NAMING': '*.example.com',
    'PLUGINS': ('ElasticBeanstalkPlugin', 'EC2Plugin')
}
```

You can use '\*' in the pattern to match any string, or '?' to match any single character\. For Flask, [configure the recorder in code](xray-sdk-python-configuration.md#xray-sdk-python-middleware-configuration-code)\.

**Example main\.py – Segment name**  

```
from aws_xray_sdk.core import xray_recorder
xray_recorder.configure(service='My application')
xray_recorder.configure(dynamic_naming='*.example.com')
```

**Note**  
You can override the default service name that you define in code with the `AWS_XRAY_TRACING_NAME` [environment variable](xray-sdk-python-configuration.md#xray-sdk-python-configuration-envvars)\.