# Tracing calls to downstream HTTP web services using the X\-Ray SDK for Python<a name="xray-sdk-python-httpclients"></a>

When your application makes calls to microservices or public HTTP APIs, you can use the X\-Ray SDK for Python to instrument those calls and add the API to the service graph as a downstream service\.

To instrument HTTP clients, [patch the library](xray-sdk-python-patching.md) that you use to make outgoing calls\. If you use `requests` or Python's built in HTTP client, that's all you need to do\. For `aiohttp`, also configure the recorder with an [async context](xray-sdk-python-patching.md#xray-sdk-python-patching-async)\.

If you use `aiohttp` 3's client API, you also need to configure the `ClientSession`'s with an instance of the tracing configuration provided by the SDK\.

**Example [`aiohttp` 3 client API](https://github.com/aws/aws-xray-sdk-python#trace-aiohttp-client-requests)**  

```
from aws_xray_sdk.ext.aiohttp.client import aws_xray_trace_config

async def foo():
    trace_config = aws_xray_trace_config()
    async with ClientSession(loop=loop, trace_configs=[trace_config]) as session:
        async with session.get(url) as resp
            await resp.read()
```

When you instrument a call to a downstream web API, the X\-Ray SDK for Python records a subsegment that contains information about the HTTP request and response\. X\-Ray uses the subsegment to generate an inferred segment for the remote API\.

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