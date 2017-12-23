# Generating Custom Subsegments with the X\-Ray SDK for Java<a name="xray-sdk-java-subsegments"></a>

A segment is a JSON document that records the work that your application does to serve a single request\. The AWSXRayServletFilter creates segments for HTTP requests and adds details about the request and response, including information from headers in the request, the time that the request was received, and the time that the response was sent\.

Further instrumentation generates *subsegments*\. Instrumented AWS SDK clients, HTTP clients, and JDBC clients add subsegments to the segment document with details of downstream calls made by the servlet or any functions that the servlet calls\.

You can create subsegments manually to organize downstream calls into groups\. For example, you can create a custom subsegment for a function that makes several calls to DynamoDB\.

**Example GameModel\.java \- Custom Subsegment**  

```
import [com\.amazonaws\.xray\.AWSXRay](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
...
  public void saveGame(Game game) throws SessionNotFoundException {
    // wrap in subsegment
    Subsegment subsegment = AWSXRay.beginSubsegment("Save Game");
    try {
      // check session
      String sessionId = game.getSession();
      if (sessionModel.loadSession(sessionId) == null ) {
        throw new SessionNotFoundException(sessionId);
      }
      mapper.save(game);
    } catch (Exception e) {
      subsegment.addException(e);
      throw e;
    } finally {
      AWSXRay.endSubsegment();
    }
  }
```

In this example, the code within the subsegment loads the game's session from DynamoDB with a method on the session model, and uses the AWS SDK for Java's DynamoDB mapper to save the game\. Wrapping this code in a subsegment makes the calls DynamoDB children of the `Save Game` subsegment in the trace view in the console\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-timeline-subsegments.png)

If the code in your subsegment throws checked exceptions, wrap it in a `try` block and call `AWSXRay.endSubsegment()` in a `finally` block to ensure that the subsegment is always closed\. If a subsegment is not closed, the parent segment cannot be completed and won't be sent to X\-Ray\.

For code that doesn't throw checked exceptions, you can pass the code to `AWSXRay.CreateSubsegment` as a Lambda function\.

**Example Subsegment Lambda Function**  

```
import [com\.amazonaws\.xray\.AWSXRay](http://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);

AWSXRay.createSubsegment("getMovies", (subsegment) -> {
    // function code
});
```

When you create a subsegment within a segment or another subsegment, the X\-Ray SDK for Java generates an ID for it and records the start time and end time\.

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