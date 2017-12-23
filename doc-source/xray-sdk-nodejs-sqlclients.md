# Tracing SQL Queries with the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-sqlclients"></a>

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