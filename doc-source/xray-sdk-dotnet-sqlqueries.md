# Tracing SQL Queries with the X\-Ray SDK for \.NET<a name="xray-sdk-dotnet-sqlqueries"></a>

The SDK provides a wrapper class for `System.Data.SqlClient.SqlCommand` named `TraceableSqlCommand` that you can use in place of `SqlCommand`\. Initialize an SQL command with the X\-Ray SDK for \.NET's `TraceableSqlCommand` class\.

**Example `Controller.cs` \- SQL Client Instrumentation**  

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

You can also execute the query asynchronously with the `ExecuteReaderAsync` method\.

**Example `Controller.cs` \- SQL Client Instrumentation \(Asynchronous\)**  

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