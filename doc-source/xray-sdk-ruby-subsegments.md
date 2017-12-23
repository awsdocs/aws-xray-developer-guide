# Generating Custom Subsegments with the X\-Ray SDK<a name="xray-sdk-ruby-subsegments"></a>

A segment is a JSON document that records the work that your application does to serve a single request\. The SDK middleware creates segments for HTTP requests and adds details about the request and response, including information from headers in the request, the time that the request was received, and the time that the response was sent\.

Further instrumentation generates *subsegments*\. Instrumented AWS SDK clients, HTTP clients, and active record clients add subsegments to the segment document with details of downstream calls made by the application\.

You can create subsegments manually to organize downstream calls into groups, or to record annotations and metadata in a separate subsegment\.

```
subsegment = XRay.recorder.begin_subsegment name: 'annotations', namespace: 'remote'
my_annotations = { id: 12345 }
subsegment.add_annotations annotations: my_annotations
XRay.recorder.end_subsegment
```

To create a subsegment for a function, wrap it in a call to `XRay.recorder.capture`\.

```
XRay.recorder.capture(name: 'a_name') do |subsegment|
  # your function here
  end
```

When you create a subsegment within a segment or another subsegment, the X\-Ray SDK generates an ID for it and records the start time and end time\.

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