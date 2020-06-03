# Add annotations and metadata to segments with the X\-Ray SDK for Java<a name="xray-sdk-java-segment"></a>

You can record additional information about requests, the environment, or your application with annotations and metadata\. You can add annotations and metadata to the segments that the X\-Ray SDK creates, or to custom subsegments that you create\.

**Annotations** are key\-value pairs with string, number, or Boolean values\. Annotations are indexed for use with [filter expressions](xray-console-filters.md)\. Use annotations to record data that you want to use to group traces in the console, or when calling the [https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html](https://docs.aws.amazon.com/xray/latest/api/API_GetTraceSummaries.html) API\.

**Metadata** are key\-value pairs that can have values of any type, including objects and lists, but are not indexed for use with filter expressions\. Use metadata to record additional data that you want stored in the trace but don't need to use with search\.

In addition to annotations and metadata, you can also [record user ID strings](#xray-sdk-java-segment-userid) on segments\. User IDs are recorded in a separate field on segments and are indexed for use with search\.

**Topics**
+ [Recording annotations with the X\-Ray SDK for Java](#xray-sdk-java-segment-annotations)
+ [Recording metadata with the X\-Ray SDK for Java](#xray-sdk-java-segment-metadata)
+ [Recording user IDs with the X\-Ray SDK for Java](#xray-sdk-java-segment-userid)

## Recording annotations with the X\-Ray SDK for Java<a name="xray-sdk-java-segment-annotations"></a>

Use annotations to record information on segments or subsegments that you want indexed for search\.

**Annotation Requirements**
+ **Keys** – Up to 500 alphanumeric characters\. No spaces or symbols except underscores\.
+ **Values** – Up to 1,000 Unicode characters\.
+ **Entries** – Up to 50 annotations per trace\.

**To record annotations**

1. Get a reference to the current segment or subsegment from `AWSXRay`\.

   ```
   import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
   import [com\.amazonaws\.xray\.entities\.Segment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Segment.html);
   ...
   Segment document = AWSXRay.getCurrentSegment();
   ```

   or

   ```
   import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
   import [com\.amazonaws\.xray\.entities\.Subsegment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Subsegment.html);
   ...
   Subsegment document = AWSXRay.getCurrentSubsegment();
   ```

1. Call `putAnnotation` with a String key, and a Boolean, Number, or String value\.

   ```
   document.putAnnotation("mykey", "my value");
   ```

The SDK records annotations as key\-value pairs in an `annotations` object in the segment document\. Calling `putAnnotation` twice with the same key overwrites previously recorded values on the same segment or subsegment\.

To find traces that have annotations with specific values, use the `annotations.key` keyword in a [filter expression](xray-console-filters.md)\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/GameModel.java](https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/GameModel.java) – Annotations and metadata**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.entities\.Segment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Segment.html);
import [com\.amazonaws\.xray\.entities\.Subsegment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Subsegment.html);
...
  public void saveGame(Game game) throws SessionNotFoundException {
    // wrap in subsegment
    Subsegment subsegment = AWSXRay.beginSubsegment("## GameModel.saveGame");
    try {
      // check session
      String sessionId = game.getSession();
      if (sessionModel.loadSession(sessionId) == null ) {
        throw new SessionNotFoundException(sessionId);
      }
      Segment segment = AWSXRay.getCurrentSegment();
      subsegment.putMetadata("resources", "game", game);
      segment.putAnnotation("gameid", game.getId());
      mapper.save(game);
    } catch (Exception e) {
      subsegment.addException(e);
      throw e;
    } finally {
      AWSXRay.endSubsegment();
    }
  }
```

## Recording metadata with the X\-Ray SDK for Java<a name="xray-sdk-java-segment-metadata"></a>

Use metadata to record information on segments or subsegments that you don't need indexed for search\. Metadata values can be strings, numbers, Booleans, or any object that can be serialized into a JSON object or array\.

**To record metadata**

1. Get a reference to the current segment or subsegment from `AWSXRay`\.

   ```
   import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
   import [com\.amazonaws\.xray\.entities\.Segment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Segment.html);
   ...
   Segment document = AWSXRay.getCurrentSegment();
   ```

   or

   ```
   import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
   import [com\.amazonaws\.xray\.entities\.Subsegment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Subsegment.html);
   ...
   Subsegment document = AWSXRay.getCurrentSubsegment();
   ```

1. Call `putMetadata` with a String namespace, String key, and a Boolean, Number, String, or Object value\.

   ```
   document.putMetadata("my namespace", "my key", "my value");
   ```

   or

   Call `putMetadata` with just a key and value\.

   ```
   document.putMetadata("my key", "my value");
   ```

If you don't specify a namespace, the SDK uses `default`\. Calling `putMetadata` twice with the same key overwrites previously recorded values on the same segment or subsegment\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/GameModel.java](https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/GameModel.java) – Annotations and metadata**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.entities\.Segment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Segment.html);
import [com\.amazonaws\.xray\.entities\.Subsegment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Subsegment.html);
...
  public void saveGame(Game game) throws SessionNotFoundException {
    // wrap in subsegment
    Subsegment subsegment = AWSXRay.beginSubsegment("## GameModel.saveGame");
    try {
      // check session
      String sessionId = game.getSession();
      if (sessionModel.loadSession(sessionId) == null ) {
        throw new SessionNotFoundException(sessionId);
      }
      Segment segment = AWSXRay.getCurrentSegment();
      subsegment.putMetadata("resources", "game", game);
      segment.putAnnotation("gameid", game.getId());
      mapper.save(game);
    } catch (Exception e) {
      subsegment.addException(e);
      throw e;
    } finally {
      AWSXRay.endSubsegment();
    }
  }
```

## Recording user IDs with the X\-Ray SDK for Java<a name="xray-sdk-java-segment-userid"></a>

Record user IDs on request segments to identify the user who sent the request\.

**To record user IDs**

1. Get a reference to the current segment from `AWSXRay`\.

   ```
   import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
   import [com\.amazonaws\.xray\.entities\.Segment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Segment.html);
   ...
   Segment document = AWSXRay.getCurrentSegment();
   ```

1. Call `setUser` with a string ID of the user who sent the request\.

   ```
   document.setUser("U12345");
   ```

You can call `setUser` in your controllers to record the user ID as soon as your application starts processing a request\. If you will only use the segment to set the user ID, you can chain the calls in a single line\.

**Example [src/main/java/scorekeep/MoveController\.java](https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/MoveController.java) – User ID**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
...
  @RequestMapping(value="/{userId}", method=RequestMethod.POST)
  public Move newMove(@PathVariable String sessionId, @PathVariable String gameId, @PathVariable String userId, @RequestBody String move) throws SessionNotFoundException, GameNotFoundException, StateNotFoundException, RulesException {
    AWSXRay.getCurrentSegment().setUser(userId);
    return moveFactory.newMove(sessionId, gameId, userId, move);
  }
```

To find traces for a user ID, use the `user` keyword in a [filter expression](xray-console-filters.md)\.