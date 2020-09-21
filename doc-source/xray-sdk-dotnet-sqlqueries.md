# Tracing SQL queries with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-sqlqueries"></a>

The X\-Ray SDK for \.NET provides a wrapper class for `System.Data.SqlClient.SqlCommand`, named `TraceableSqlCommand`, that you can use in place of `SqlCommand`\. You can initialize an SQL command with the `TraceableSqlCommand` class\.

## Tracing SQL queries with synchronous and asynchronous methods<a name="xray-sdk-dotnot-sqlqueries-trace"></a>

The following examples show how to use the `TraceableSqlCommand` to automatically trace SQL Server queries synchronously and asynchronously\.

**Example `Controller.cs` \- SQL client instrumentation \(synchronous\)**  

```
using Amazon;
using Amazon.Util;
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
using [Amazon\.XRay\.Recorder\.Handlers\.SqlServer](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_SqlServer.htm);

private void QuerySql(int id)
{
  var connectionString = ConfigurationManager.AppSettings["RDS_CONNECTION_STRING"];
  using (var sqlConnection = new SqlConnection(connectionString))
  using (var sqlCommand = new TraceableSqlCommand("SELECT " + id, sqlConnection))
  {
    sqlCommand.Connection.Open();
    sqlCommand.ExecuteNonQuery();
  }
}
```

You can execute the query asynchronously by using the `ExecuteReaderAsync` method\.

**Example `Controller.cs` \- SQL client instrumentation \(asynchronous\)**  

```
using Amazon;
using Amazon.Util;
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
using [Amazon\.XRay\.Recorder\.Handlers\.SqlServer](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_SqlServer.htm);
private void QuerySql(int id)
{
  var connectionString = ConfigurationManager.AppSettings["RDS_CONNECTION_STRING"];
  using (var sqlConnection = new SqlConnection(connectionString))
  using (var sqlCommand = new TraceableSqlCommand("SELECT " + id, sqlConnection))
  {
    await sqlCommand.ExecuteReaderAsync();
  }
}
```

## Collecting SQL queries made to SQL Server<a name="xray-sdk-dotnot-sqlqueries-collect"></a>

You can enable the capture of `SqlCommand.CommandText` as part of the subsegment created by your SQL query\. `SqlCommand.CommandText` appears as the field `sanitized_query` in the subsegment JSON\. By default, this feature is disabled for security\. 

**Note**  
Do not enable the collection feature if you are including sensitive information as clear text in your SQL queries\.

You can enable the collection of SQL queries in two ways: 
+ Set the `CollectSqlQueries` property to `true` in the global configuration for your application\.
+ Set the `collectSqlQueries` parameter in the `TraceableSqlCommand` instance to `true` to collect calls within the instance\.

### Enable the global CollectSqlQueries property<a name="xray-sdk-dotnot-sqlqueries-collect-global"></a>

The following examples show how to enable the `CollectSqlQueries` property for \.NET and \.NET Core\.

------
#### [ \.NET ]

To set the `CollectSqlQueries` property to `true` in the global configuration of your application in \.NET, modify the `appsettings` of your `App.config` or `Web.config` file, as shown\.

**Example `App.config` Or `Web.config` – Enable SQL Query collection globally**  

```
<configuration>
<appSettings>
    <add key="CollectSqlQueries" value="true">
</appSettings>
</configuration>
```

------
#### [ \.NET Core ]

To set the `CollectSqlQueries` property to `true` in the global configuration of your application in \.NET Core, modify your `appsettings.json` file under the X\-Ray key, as shown\.

**Example `appsettings.json` – Enable SQL Query collection globally**  

```
{
  "XRay": {
    "CollectSqlQueries":"true"
  }
}
```

------

### Enable the collectSqlQueries parameter<a name="xray-sdk-dotnot-sqlqueries-collect-instance"></a>

You can set the `collectSqlQueries` parameter in the `TraceableSqlCommand` instance to `true` to collect the SQL query text for SQL Server queries made using that instance\. Setting the parameter to `false` disables the `CollectSqlQuery` feature for the `TraceableSqlCommand` instance\. 

**Note**  
 The value of `collectSqlQueries` in the `TraceableSqlCommand` instance overrides the value set in the global configuration of the `CollectSqlQueries` property\.

**Example `Controller.cs` – Enable SQL Query collection for the instance**  

```
using Amazon;
using Amazon.Util;
using [Amazon\.XRay\.Recorder\.Core](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Core.htm);
using [Amazon\.XRay\.Recorder\.Handlers\.SqlServer](https://docs.aws.amazon.com/xray-sdk-for-dotnet/latest/reference/html/N_Amazon_XRay_Recorder_Handlers_SqlServer.htm);

private void QuerySql(int id)
{
  var connectionString = ConfigurationManager.AppSettings["RDS_CONNECTION_STRING"];
  using (var sqlConnection = new SqlConnection(connectionString))
  using (var command = new TraceableSqlCommand("SELECT " + id, sqlConnection, collectSqlQueries: true))
  {
    command.ExecuteNonQuery();
  }
}
```