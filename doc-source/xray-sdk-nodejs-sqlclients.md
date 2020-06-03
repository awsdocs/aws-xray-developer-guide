# Tracing SQL queries with the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-sqlclients"></a>

Instrument SQL database queries by wrapping your SQL client in the corresponding X\-Ray SDK for Node\.js client method\.
+  **PostgreSQL** – `AWSXRay.capturePostgres()` 

  ```
  var AWSXRay = require('aws-xray-sdk');
  var pg = AWSXRay.capturePostgres(require('pg'));
  var client = new pg.Client();
  ```
+  **MySQL** – `AWSXRay.captureMySQL()` 

  ```
  var AWSXRay = require('aws-xray-sdk');
  var mysql = AWSXRay.captureMySQL(require('mysql'));
  ...
  var connection = mysql.createConnection(config);
  ```

When you use an instrumented client to make SQL queries, the X\-Ray SDK for Node\.js records information about the connection and query in a subsegment\.

## Including additional data in SQL subsegments<a name="xray-sdk-nodejs-sqlclients-additional"></a>

You can add additional information to subsegments generated for SQL queries, as long as it's mapped to a whitelisted SQL field\. For example, to record the sanitized SQL query string in a subsegment, you can add it directly to the subsegment's SQL object\.

**Example Assign SQL to sugsegment**  

```
    const queryString = 'SELECT * FROM MyTable';
connection.query(queryString, ...);

// Retrieve the most recently created subsegment
const subs = AWSXRay.getSegment().subsegments;

if (subs & & subs.length > 0) {
  var sqlSub = subs[subs.length - 1];
  sqlSub.sql.sanitized_query = queryString;
}
```

For a full list of whitelisted SQL fields, see [SQL Queries](https://docs.aws.amazon.com/xray/latest/devguide/xray-api-segmentdocuments.html#api-segmentdocuments-sql) in the *AWS X\-Ray Developer Guide*\.