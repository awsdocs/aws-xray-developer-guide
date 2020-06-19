# Manually instrumenting AWS SDK clients<a name="scorekeep-sdkclients"></a>

The X\-Ray SDK for Java automatically instruments all AWS SDK clients when you [include the AWS SDK Instrumentor submodule in your build dependencies](xray-sdk-java.md#xray-sdk-java-dependencies)\.

You can disable automatic client instrumentation by removing the Instrumentor submodule\. This enables you to instrument some clients manually while ignoring others, or use different tracing handlers on different clients\.

To illustrate support for instrumenting specific AWS SDK clients, the application passes a tracing handler to `AmazonDynamoDBClientBuilder` as a request handler in the user, game, and session model\. This code change tells the SDK to instrument all calls to DynamoDB using those clients\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/SessionModel.java](https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/SessionModel.java) – Manual AWS SDK client instrumentation**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.handlers\.TracingHandler](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/handlers/TracingHandler.html);

public class SessionModel {
  private AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
        .withRegion(Constants.REGION)
        .withRequestHandlers(new TracingHandler(AWSXRay.getGlobalRecorder()))
        .build();
  private DynamoDBMapper mapper = new DynamoDBMapper(client);
```

If you remove the AWS SDK Instrumentor submodule from project dependencies, only the manually instrumented AWS SDK clients appear in the service map\.