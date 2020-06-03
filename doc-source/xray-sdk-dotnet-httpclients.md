# Tracing calls to downstream HTTP web services with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-httpclients"></a>

When your application makes calls to microservices or public HTTP APIs, you can use the X\-Ray SDK for \.NET's `GetResponseTraced` extension method for `System.Net.HttpWebRequest` to instrument those calls and add the API to the service graph as a downstream service\.

**Example HttpWebRequest**  

```
using System.Net;
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
using [Amazon\.XRay\.Recorder\.Handlers\.System\.Net](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_System_Net.htm);

private void MakeHttpRequest()
{
  HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://names.example.com/api");
  request.GetResponseTraced();
}
```

For asynchronous calls, use `GetAsyncResponseTraced`\.

```
request.GetAsyncResponseTraced();
```

If you use [https://msdn.microsoft.com/en-us/library/system.net.http.httpclient.aspx](https://msdn.microsoft.com/en-us/library/system.net.http.httpclient.aspx), use the `HttpClientXRayTracingHandler` delegating handler to record calls\.

**Example HttpClient**  

```
using System.Net.Http;
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
using [Amazon\.XRay\.Recorder\.Handlers\.System\.Net](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_System_Net.htm);

private void MakeHttpRequest()
{
  var httpClient = new HttpClient(new HttpClientXRayTracingHandler(new HttpClientHandler()));
  httpClient.GetAsync(URL);
}
```

When you instrument a call to a downstream web API, the X\-Ray SDK for \.NET records a subsegment with information about the HTTP request and response\. X\-Ray uses the subsegment to generate an inferred segment for the API\.

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