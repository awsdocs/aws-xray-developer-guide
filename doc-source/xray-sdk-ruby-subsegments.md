# Generating custom subsegments with the X\-Ray SDK<a name="xray-sdk-ruby-subsegments"></a>

Subsegments extend a trace's [segment](xray-concepts.md#xray-concepts-segments) with details about work done in order to serve a request\. Each time you make a call with an instrumented client, the X\-Ray SDK records the information generated in a subsegment\. You can create additional subsegments to group other subsegments, to measure the performance of a section of code, or to record annotations and metadata\.

To manage subsegments, use the `begin_subsegment` and `end_subsegment` methods\.

```
subsegment = XRay.recorder.begin_subsegment name: 'annotations', namespace: 'remote'
my_annotations = { id: 12345 }
subsegment.annotations.update my_annotations
XRay.recorder.end_subsegment
```

To create a subsegment for a function, wrap it in a call to `XRay.recorder.capture`\.

```
XRay.recorder.capture('name_for_subsegment') do |subsegment|
  resp = myfunc() # myfunc is your function
  subsegment.annotations.update k1: 'v1'
  resp
end
```

When you create a subsegment within a segment or another subsegment, the X\-Ray SDK generates an ID for it and records the start time and end time\.

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