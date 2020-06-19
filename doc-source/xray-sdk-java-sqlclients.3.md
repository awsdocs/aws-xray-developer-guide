# Tracing SQL queries with the X\-Ray SDK for Java<a name="xray-sdk-java-sqlclients"></a>

Instrument SQL database queries by adding the X\-Ray SDK for Java JDBC interceptor to your data source configuration\.
+  **PostgreSQL** – `com.amazonaws.xray.sql.postgres.TracingInterceptor` 
+  **MySQL** – `com.amazonaws.xray.sql.mysql.TracingInterceptor` 

These interceptors are in the [`aws-xray-recorder-sql-postgres` and `aws-xray-recorder-sql-mysql` submodules](xray-sdk-java.md), respectively\. They implement `org.apache.tomcat.jdbc.pool.JdbcInterceptor` and are compatible with Tomcat connection pools\.

**Note**  
SQL interceptors do not record the SQL query itself within subsegments for security purposes\.

For Spring, add the interceptor in a properties file and build the data source with Spring Boot's `DataSourceBuilder`\.

**Example `src/main/java/resources/application.properties` \- PostgreSQL JDBC interceptor**  

```
spring.datasource.continue-on-error=true
spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=create-drop
spring.datasource.jdbc-interceptors=com.amazonaws.xray.sql.postgres.TracingInterceptor
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQL94Dialect
```

**Example `src/main/java/myapp/WebConfig.java` \- Data source**  

```
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

import javax.servlet.Filter;
import javax.sql.DataSource;
import java.net.URL;

@Configuration
@EnableAutoConfiguration
@EnableJpaRepositories("myapp")
public class RdsWebConfig {

  @Bean
  @ConfigurationProperties(prefix = "spring.datasource")
  public DataSource dataSource() {
      logger.info("Initializing PostgreSQL datasource");
      return DataSourceBuilder.create()
              .driverClassName("org.postgresql.Driver")
              .url("jdbc:postgresql://" + System.getenv("RDS_HOSTNAME") + ":" + System.getenv("RDS_PORT") + "/ebdb")
              .username(System.getenv("RDS_USERNAME"))
              .password(System.getenv("RDS_PASSWORD"))
              .build();
  }
...
}
```

For Tomcat, call `setJdbcInterceptors` on the JDBC data source with a reference to the X\-Ray SDK for Java class\.

**Example `src/main/myapp/model.java` \- Data source**  

```
import org.apache.tomcat.jdbc.pool.DataSource;
...
DataSource source = new DataSource();
source.setUrl(url);
source.setUsername(user);
source.setPassword(password);
source.setDriverClassName("com.mysql.jdbc.Driver");
source.setJdbcInterceptors("com.amazonaws.xray.sql.mysql.TracingInterceptor;");
```

The Tomcat JDBC Data Source library is included in the X\-Ray SDK for Java, but you can declare it as a provided dependency to document that you use it\.

**Example `pom.xml` \- JDBC data source**  

```
<dependency>
  <groupId>org.apache.tomcat</groupId>
  <artifactId>tomcat-jdbc</artifactId>
  <version>8.0.36</version>
  <scope>provided</scope>
</dependency>
```