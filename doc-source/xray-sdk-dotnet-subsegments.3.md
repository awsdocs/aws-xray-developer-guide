# Creating additional subsegments<a name="xray-sdk-dotnet-subsegments"></a>

Subsegments extend a trace's [segment](xray-concepts.md#xray-concepts-segments) with details about work done in order to serve a request\. Each time you make a call with an instrumented client, the X\-Ray SDK records the information generated in a subsegment\. You can create additional subsegments to group other subsegments, to measure the performance of a section of code, or to record annotations and metadata\.

To manage subsegments, use the `BeginSubsegment` and `EndSubsegment` methods\. Perform any work in the subsegment in a `try` block and use `AddException` to trace exceptions\. Call `EndSubsegment` in a `finally` block to ensure that the subsegment is closed\.

**Example Controller\.cs – Custom subsegment**  

```
AWSXRayRecorder.Instance.BeginSubsegment("custom method");
try
{
  DoWork();
}
catch (Exception e)
{
  AWSXRayRecorder.Instance.AddException(e);
}
finally
{
  AWSXRayRecorder.Instance.EndSubsegment();
}
```

When you create a subsegment within a segment or another subsegment, the X\-Ray SDK for \.NET generates an ID for it and records the start time and end time\.

**Example Subsegment with metadata**  

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