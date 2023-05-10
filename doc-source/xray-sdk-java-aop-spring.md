# AOP with Spring and the X\-Ray SDK for Java<a name="xray-sdk-java-aop-spring"></a>

This topic describes how to use the X\-Ray SDK and the Spring Framework to instrument your application without changing its core logic\. This means that there is now a non\-invasive way to instrument your applications running remotely in AWS\.

**To enable AOP in spring**

1. [Configure Spring](#xray-sdk-java-aop-spring-configuration)

1. [Add a tracing filter to your application](#xray-sdk-java-aop-filters-spring)

1. [Annotate your code or implement an interface](#xray-sdk-java-aop-annotate-or-implement)

1. [Activate X\-Ray in your application](#xray-sdk-java-aop-activate-xray)

## Configuring Spring<a name="xray-sdk-java-aop-spring-configuration"></a>

You can use Maven or Gradle to configure Spring to use AOP to instrument your application\.

If you use Maven to build your application, add the following dependency in your `pom.xml` file\.

```
<dependency> 
     <groupId>com.amazonaws</groupId> 
     <artifactId>aws-xray-recorder-sdk-spring</artifactId> 
     <version>2.11.0</version> 
</dependency>
```

For Gradle, add the following dependency in your `build.gradle` file\.

```
compile 'com.amazonaws:aws-xray-recorder-sdk-spring:2.11.0'
```

## Configuring Spring Boot<a name="xray-sdk-java-aop-spring-boot-configuration"></a>

In addition to the Spring dependency described in the previous section, if you’re using Spring Boot, add the following dependency if it’s not already on your classpath\. 

Maven:

```
<dependency> 
     <groupId>org.springframework.boot</groupId> 
     <artifactId>spring-boot-starter-aop</artifactId> 
     <version>2.5.2</version> 
</dependency>
```

Gradle:

```
compile 'org.springframework.boot:spring-boot-starter-aop:2.5.2'
```

## Adding a tracing filter to your application<a name="xray-sdk-java-aop-filters-spring"></a>

Add a `Filter` to your `WebConfig` class\. Pass the segment name to the [https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html) constructor as a string\. For more information about tracing filters and instrumenting incoming requests, see [Tracing incoming requests with the X\-Ray SDK for Java](xray-sdk-java-filters.md)\.

**Example src/main/java/myapp/WebConfig\.java \- spring**  

```
package myapp;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import javax.servlet.Filter;
import [com\.amazonaws\.xray\.javax\.servlet\.AWSXRayServletFilter](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/javax/servlet/AWSXRayServletFilter.html);

@Configuration
public class WebConfig {

  @Bean
  public Filter TracingFilter() {
    return new AWSXRayServletFilter("Scorekeep");
  }
}
```

## Jakarta Support<a name="xray-sdk-java-aop-jakarta-support"></a>

 Spring 6 uses [Jakarta](https://spring.io/blog/2022/11/16/spring-framework-6-0-goes-ga) instead of Javax for its Enterprise Edition\. To support this new namespace, X\-Ray has created a parallel set of classes that live in their own Jakarta namespace\. 

For the filter classes, replace `javax` with `jakarta`\. When configuring a segment naming strategy, add `jakarta` before the naming strategy class name, as in the following example:

```
package myapp;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import jakarta.servlet.Filter;
import com.amazonaws.xray.jakarta.servlet.AWSXRayServletFilter;
import com.amazonaws.xray.strategy.jakarta.SegmentNamingStrategy;

@Configuration
public class WebConfig {
    @Bean
    public Filter TracingFilter() {
        return new AWSXRayServletFilter(SegmentNamingStrategy.dynamic("Scorekeep"));
    }
}
```

## Annotating your code or implementing an interface<a name="xray-sdk-java-aop-annotate-or-implement"></a>

Your classes must either be annotated with the `@XRayEnabled` annotation, or implement the `XRayTraced` interface\. This tells the AOP system to wrap the functions of the affected class for X\-Ray instrumentation\.

## Activating X\-Ray in your application<a name="xray-sdk-java-aop-activate-xray"></a>

To activate X\-Ray tracing in your application, your code must extend the abstract class `BaseAbstractXRayInterceptor` by overriding the following methods\.
+ `generateMetadata`—This function allows customization of the metadata attached to the current function’s trace\. By default, the class name of the executing function is recorded in the metadata\. You can add more data if you need additional information\.
+ `xrayEnabledClasses`—This function is empty, and should remain so\. It serves as the host for a pointcut instructing the interceptor about which methods to wrap\. Define the pointcut by specifying which of the classes that are annotated with `@XRayEnabled` to trace\. The following pointcut statement tells the interceptor to wrap all controller beans annotated with the `@XRayEnabled` annotation\.

  ```
  @Pointcut(“@within(com.amazonaws.xray.spring.aop.XRayEnabled) && bean(*Controller)”)
  ```

 If your project is using Spring Data JPA, consider extending from `AbstractXRayInterceptor` instead of `BaseAbstractXRayInterceptor`\. 

## Example<a name="xray-sdk-java-aop-example"></a>

The following code extends the abstract class `BaseAbstractXRayInterceptor`\.

```
@Aspect
@Component
public class XRayInspector extends BaseAbstractXRayInterceptor {    
    @Override    
    protected Map<String, Map<String, Object>> generateMetadata(ProceedingJoinPoint proceedingJoinPoint, Subsegment subsegment) throws Exception {      
        return super.generateMetadata(proceedingJoinPoint, subsegment);    
    }    
  
  @Override    
  @Pointcut("@within(com.amazonaws.xray.spring.aop.XRayEnabled) && bean(*Controller)")    
  public void xrayEnabledClasses() {}
  
}
```

The following code is a class that will be instrumented by X\-Ray\.

```
@Service
@XRayEnabled
public class MyServiceImpl implements MyService {    
    private final MyEntityRepository myEntityRepository;    
    
    @Autowired    
    public MyServiceImpl(MyEntityRepository myEntityRepository) {        
        this.myEntityRepository = myEntityRepository;    
    }    
    
    @Transactional(readOnly = true)    
    public List<MyEntity> getMyEntities(){        
        try(Stream<MyEntity> entityStream = this.myEntityRepository.streamAll()){            
            return entityStream.sorted().collect(Collectors.toList());        
        }    
    }
}
```

If you've configured your application correctly, you should see the complete call stack of the application, from the controller down through the service calls, as shown in the following screen shot of the console\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/aop-spring-console.png)