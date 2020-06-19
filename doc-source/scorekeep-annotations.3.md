# Recording annotations, metadata, and user IDs<a name="scorekeep-annotations"></a>

In the game model class, the application records `Game` objects in a [metadata](xray-sdk-java-segment.md#xray-sdk-java-segment-metadata) block each time it saves a game in DynamoDB\. Separately, the application records game IDs in [annotations](xray-sdk-java-segment.md#xray-sdk-java-segment-annotations) for use with [filter expressions](xray-console-filters.md)\.

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

In the move controller, the application records [user IDs](xray-sdk-java-segment.md#xray-sdk-java-segment-userid) with `setUser`\. User IDs are recorded in a separate field on segments and are indexed for use with search\.

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