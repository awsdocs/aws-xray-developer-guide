# Tracing calls to downstream HTTP web services using the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-httpclients"></a>

When your application makes calls to microservices or public HTTP APIs, you can use the X\-Ray SDK for Node\.js client to instrument those calls and add the API to the service graph as a downstream service\.

Pass your `http` or `https` client to the X\-Ray SDK for Node\.js `captureHTTPs` method to trace outgoing calls\.

**Note**  
Calls using third\-party HTTP request libraries, such as Axios or Superagent, are supported through the [`captureHTTPsGlobal()` API](https://docs.aws.amazon.com/xray-sdk-for-nodejs/latest/reference/module-http_p.html) and will still be traced when they use the native `http` module\.

**Example app\.js \- HTTP client**  

```
var AWSXRay = require('aws-xray-sdk');
var http = AWSXRay.captureHTTPs(require('http'));
```

To enable tracing on all HTTP clients, call `captureHTTPsGlobal` before you load `http`\.

**Example app\.js \- HTTP client \(global\)**  

```
var AWSXRay = require('aws-xray-sdk');
AWSXRay.captureHTTPsGlobal(require('http'));
var http = require('http');
```

When you instrument a call to a downstream web API, the X\-Ray SDK for Node\.js records a subsegment that contains information about the HTTP request and response\. X\-Ray uses the subsegment to generate an inferred segment for the remote API\.

**Example Subsegment for a downstream HTTP call**  

```
{
  "id": "004f72be19cddc2a",
  "start_time": 1484786387.131,
  "end_time": 1484786387.501,
  "name": "names.example.com",
  "namespace": "remote",
  "http": {
    "request": {
      "method": "GET",
      "url": "https://names.example.com/"
    },
    "response": {
      "content_length": -1,
      "status": 200
    }
  }
}
```

**Example Inferred segment for a downstream HTTP call**  

```
{
  "id": "168416dc2ea97781",
  "name": "names.example.com",
  "trace_id": "1-5880168b-fd5153bb58284b67678aa78c",
  "start_time": 1484786387.131,
  "end_time": 1484786387.501,
  "parent_id": "004f72be19cddc2a",
  "http": {
    "request": {
      "method": "GET",
      "url": "https://names.example.com/"
    },
    "response": {
      "content_length": -1,
      "status": 200
    }
  },
  "inferred": true
}
```