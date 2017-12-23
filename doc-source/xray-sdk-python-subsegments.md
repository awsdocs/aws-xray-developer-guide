# Generating Custom Subsegments with the X\-Ray SDK for Python<a name="xray-sdk-python-subsegments"></a>

A segment is a JSON document that records the work that your application does to serve a single request\. The SDK for Python middleware creates segments for HTTP requests and adds details about the request and response\. On AWS Lambda, Lambda creates the segment\. Details include information from headers in the request, the time that the request was received, and the time that the response was sent\.

Further instrumentation generates *subsegments*\. Instrumented AWS SDK clients, HTTP clients, and SQL clients add subsegments to the segment document with details of downstream calls made by the servlet or any functions that the servlet calls\.

You can create subsegments manually to organize downstream calls into groups, or to record annotations and metadata in a separate subsegment\.

**Example main\.py – custom subsegment**  

```
from aws_xray_sdk.core import xray_recorder

subsegment = xray_recorder.begin_subsegment('annotations')
subsegment.put_annotation('id', 12345)
xray_recorder.end_subsegment()
```

To create a subsegment for a synchronous function, use the `@xray_recorder.capture` decorator\. You can pass a name for the subsegment to the capture function or leave it out to use the function name\.

**Example main\.py – function subsegment**  

```
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('## create_user')
def create_user():
...
```

For an asynchronous function, use the `@xray_recorder.capture_async` decorator, and pass an async context to the recorder\.

**Example main\.py – asynchronous function subsegment**  

```
from aws_xray_sdk.core.async_context import AsyncContext
from aws_xray_sdk.core import xray_recorder
xray_recorder.configure(service='my_service', context=AsyncContext())

@xray_recorder.capture_async('## create_user')
async def create_user():
    ...

async def main():
    await myfunc()
```

When you create a subsegment within a segment or another subsegment, the X\-Ray SDK for Python generates an ID for it and records the start time and end time\.

**Example Subsegment with Metadata**  

```
"subsegments": [{
  "id": "6f1605cd8a07cb70",
  "start_time": 1.480305974194E9,
  "end_time": 1.4803059742E9,
  "name": "Custom subsegment for UserModel.saveUser function",
  "metadata": {
    "debug": {
      "test": "Metadata string from UserModel.saveUser"
    }
  },
```