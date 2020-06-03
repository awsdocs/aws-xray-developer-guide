# Creating additional subsegments<a name="scorekeep-subsegments"></a>

In the user model class, the application manually creates subsegments to group all downstream calls made within the `saveUser` function and adds metadata\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/UserModel.java](https://github.com/awslabs/eb-java-scorekeep/tree/xray/src/main/java/scorekeep/UserModel.java) \- Custom subsegments**  

```
import [com\.amazonaws\.xray\.AWSXRay](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/AWSXRay.html);
import [com\.amazonaws\.xray\.entities\.Subsegment](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/entities/Subsegment.html);
...
    public void saveUser(User user) {
    // Wrap in subsegment
    Subsegment subsegment = AWSXRay.beginSubsegment("## UserModel.saveUser");
    try {
      mapper.save(user);
    } catch (Exception e) {
      subsegment.addException(e);
      throw e;
    } finally {
      AWSXRay.endSubsegment();
    }
  }
```