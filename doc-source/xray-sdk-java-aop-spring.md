# AOP with Spring and the X\-Ray SDK for Java<a name="xray-sdk-java-aop-spring"></a>

This topic describes how to use the X\-Ray SDK and the Spring Framework to instrument your application without changing its core logic\. This means that there is now a non\-invasive way to instrument your applications running remotely in AWS\.

You must perform three tasks to enable this feature\.

**To enable AOP in spring**

1. [Configure Spring](#xray-sdk-java-aop-spring-configuration)

1. [Annotate your code or implement an interface](#xray-sdk-java-aop-annotate-or-implement)

1. [Activate X\-Ray in your application](#xray-sdk-java-aop-activate-xray)

## Configuring Spring<a name="xray-sdk-java-aop-spring-configuration"></a>

You can use Maven or Gradle to configure Spring to use AOP to instrument your application\.

If you use Maven to build your application, add the following dependency in your `pom.xml` file\.

```
<dependency> 
     <groupId>com.amazonaws</groupId> 
     <artifactId>aws-xray-recorder-sdk-spring</artifactId> 
     <version>2.4.0</version> 
</dependency>
```

For Gradle, add the following dependency in your `build.gradle` file\.

```
compile 'com.amazonaws:aws-xray-recorder-sdk-spring:2.4.0'
```

## Annotating your code or implementing an interface<a name="xray-sdk-java-aop-annotate-or-implement"></a>

Your classes must either be annotated with the `@XRayEnabled` annotation, or implement the `XRayTraced` interface\. This tells the AOP system to wrap the functions of the affected class for X\-Ray instrumentation\.

## Activating X\-Ray in your application<a name="xray-sdk-java-aop-activate-xray"></a>

To activate X\-Ray tracing in your application, your code must extend the abstract class `AbstractXRayInterceptor` by overriding the following methods\.
+ `generateMetadata`—This function allows customization of the metadata attached to the current function’s trace\. By default, the class name of the executing function is recorded in the metadata\. You can add more data if you need additional information\.
+ `xrayEnabledClasses`—This function is empty, and should remain so\. It serves as the host for a pointcut instructing the interceptor about which methods to wrap\. Define the pointcut by specifying which of the classes that are annotated with `@XRayEnabled` to trace\. The following pointcut statement tells the interceptor to wrap all controller beans annotated with the `@XRayEnabled` annotation\.

  ```
  @Pointcut(“@within(com.amazonaws.xray.spring.aop.XRayEnabled) && bean(*Controller)”)
  ```

## Example<a name="xray-sdk-java-aop-example"></a>

The following code extends the abstract class `AbstractXRayInterceptor`\.

```
@Aspect
@Component
public class XRayInspector extends AbstractXRayInterceptor {    
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