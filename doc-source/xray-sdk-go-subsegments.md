# Generating Custom Subsegments with the X\-Ray SDK for Go<a name="xray-sdk-go-subsegments"></a>

A segment is a JSON document that records the work that your application does to serve a single request\. The handler method creates segments for HTTP requests and adds details about the request and response, including information from headers in the request, the time that the request was received, and the time that the response was sent\.

Further instrumentation generates *subsegments*\. Instrumented AWS SDK clients and HTTP clients add subsegments to the segment document with details of downstream calls made by your application while the request segment is open\.

You can create subsegments manually to organize downstream calls into groups\. For example, you can create a custom subsegment for a function that makes several calls to DynamoDB\.

In the following example, the code creates a subsegment for the `GameModel.saveGame` function in a Go application\.

**Example main\.go – custom subsegment**  

```
func criticalSection(ctx context.Context) {
  //this is an example of a subsegment
  xray.Capture(ctx, "GameModel.saveGame", func(ctx1 context.Context) error {
    var err error

    section.Lock()
    result := someLockedResource.Go()
    section.Unlock()

    xray.AddMetadata(ctx1, "ResourceResult", result)
  })
```

The following screenshot shows an example of how the `saveGame` subsegment might appear in traces for the application `Scorekeep`\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-timeline-subsegments.png)