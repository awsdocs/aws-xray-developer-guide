# AWS Elastic Beanstalk and AWS X\-Ray<a name="xray-services-beanstalk"></a>

AWS Elastic Beanstalk platforms include the X\-Ray daemon\. You can [run the daemon](xray-daemon-beanstalk.md) by setting an option in the Elastic Beanstalk console or with a configuration file\.

On the Java SE platform, you can use a Buildfile file to build your application with Maven or Gradle on\-instance\. The X\-Ray SDK for Java and AWS SDK for Java are available from Maven, so you can deploy only your application code and build on\-instance to avoid bundling and uploading all of your dependencies\.

You can use Elastic Beanstalk environment properties to configure the X\-Ray SDK\. The method that Elastic Beanstalk uses to pass environment properties to your application varies by platform\. Use the X\-Ray SDK's environment variables or system properties depending on your platform\.
+ **[Node\.js platform](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_nodejs.container.html)** – Use [environment variables](xray-sdk-nodejs-configuration.md#xray-sdk-nodejs-configuration-envvars)
+ **[Java SE platform](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-platform.html)** – Use [environment variables](xray-sdk-java-configuration.md#xray-sdk-java-configuration-envvars)
+ **[Tomcat platform](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-tomcat-platform.html)** – Use [system properties](xray-sdk-java-configuration.md#xray-sdk-java-configuration-sysprops)

For more information, see [Configuring AWS X\-Ray Debugging](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-debugging.html) in the AWS Elastic Beanstalk Developer Guide\.