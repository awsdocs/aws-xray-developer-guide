# Searching for Traces in the AWS X\-Ray Console with Filter Expressions<a name="xray-console-filters"></a>

When you choose a time period of traces to view in the X\-Ray console, you might get more results than the console can display\. In the top\-right corner, the console shows the number of traces that it scanned and, whether there are more traces available\. You can narrow the results to just the traces that you want to find by using a **filter expression**\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-tracescan-max.png)

When you [choose a node in the service map](xray-console.md#xray-console-servicemap), the console constructs a filter expression based on the service name of the node, and the types of error present based on your selection\. To find traces that show performance issues or that relate to specific requests, you can adjust the expression provided by the console, or create your own\. If you add annotations with the X\-Ray SDK, you can also filter based on the presence of an annotation key or the value of a key\.

**Note**  
If you choose a relative time range in the service map, the console converts it to an absolute start and end time when you choose a node\. To ensure that the traces for the node appear in the search results, and avoid scanning times when the node was not active, the time range only includes times when the node sent traces\. If you want to search relative to the current time, you can switch back to a relative time range in the traces page and re\-scan\.

If there are still more results available than the console can show, the console shows you how many traces matched and the number of traces scanned\. The percentage shown is the percentage of the selected time frame that was scanned\. Narrow your filter expression further, or choose a shorter time frame, to ensure that you see all matching traces represented in the results\.

To get the freshest results first, the console starts scanning at the end of the time range and works backwards\. If there are a large number of traces, but few results, the console splits the time range into chunks and scans them in parallel\. The progress bar shows the parts of the time range that have been scanned\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-tracescan-parallel.png)

**Topics**
+ [Filter Expression Syntax](#console-filters-syntax)
+ [Boolean Keywords](#console-filters-boolean)
+ [Number Keywords](#console-filters-number)
+ [String Keywords](#console-filters-string)
+ [Complex Keywords](#console-filters-complex)
+ [The ID Function](#console-filters-functions)

## Filter Expression Syntax<a name="console-filters-syntax"></a>

Filter expressions can contain a *keyword*, a unary or binary *operator*, and a *value* for comparison\.

```
keyword operator value
```

Different operators are available for different types of keyword\. For example, `responsetime` is a number keyword and can be compared with operators related to numbers\.

**Example Requests where response time was more than 5 seconds**  

```
responsetime > 5
```

You can combine multiple expressions in a compound expression with the `AND` and `OR` operators\.

**Example Requests where the total duration was 5 to 8 seconds**  

```
duration >= 5 AND duration <= 8
```

Simple keywords and operators only find issues at the trace level\. If an error occurs downstream, but is handled by your application and not returned to the user, a search for `error` will not find it\.

To find traces with downstream issues, you can use the [complex keywords](#console-filters-complex) `service()` and `edge()`\. These keywords let you apply a filter expression to all downstream nodes, a single downstream node, or an edge between two nodes\. For even more granularity, you can filter services and edges by type with [the id\(\) function](#console-filters-functions)\.

## Boolean Keywords<a name="console-filters-boolean"></a>

Boolean keywords are either true or false\. Use these keywords to find traces that resulted in errors\.

**Boolean Keywords**
+ `ok` – Response status code was 2XX Success\.
+ `error` – Response status code was 4XX Client Error\.
+ `throttle` – Response status code was *429 Too Many Requests*\.
+ `fault` – Response status code was 5XX Server Error\.
+ `partial` – Request has incomplete segments\.

Boolean operators find segments where the specified key is `true` or `false`\.

**Boolean Operators**
+ none – The expression is true if the keyword is true\.
+ `!` – The expression is true if the keyword is false\.
+ `=`,`!=` – Compare the value of the keyword to the string `true` or `false`\. Acts the same as the other operators but is more explicit\.

**Example Response status is 2XX OK**  

```
ok
```

**Example Response status is not 2XX OK**  

```
!ok
```

**Example Response status is not 2XX OK**  

```
ok = false
```

## Number Keywords<a name="console-filters-number"></a>

Number keywords let you search for requests with a specific response time, duration, or response status\.

**Number Keywords**
+ `responsetime` – Time that the server took to send a response\.
+ `duration` – Total request duration, including all downstream calls\.
+ `http.status` – Response status code\.

Number keywords use standard equality and comparison operators\.

**Number Operators**
+ `=`,`!=` – The keyword is equal to or not equal to a number value\.
+ `<`,`<=`, `>`,`>=` – The keyword is less than or greater than a number value\.

**Example Response status is not 200 OK**  

```
http.status != 200
```

**Example Request where the total duration was 5 to 8 seconds**  

```
duration >= 5 AND duration <= 8
```

**Example Requests that completed successfully in under 3 seconds, including all downstream calls**  

```
ok !partial duration <3
```

## String Keywords<a name="console-filters-string"></a>

String keywords let you find traces with specific text in the request headers, or user IDs\.

**String Keywords**
+ `http.url` – Request URL\.
+ `http.method` – Request method\.
+ `http.useragent` – Request user agent string\.
+ `http.clientip` – Requestor's IP address\.
+ `user` – Value of user field on any segment in the trace\.

String operators find values that are equal to or contain specific text\. Values must always be specified in quotation marks\. 

**String Operators**
+ `=`,`!=` – The keyword is equal to or not equal to a number value\.
+ `CONTAINS` – The keyword contains a specific string\.
+ `BEGINSWITH` ,`ENDSWITH` – The keyword starts or ends with a specific string\.

**Example User filter**  

```
http.url CONTAINS "/api/game/"
```

To test if a field exists on a trace, regardless of its value, check to see if it contains the empty string\.

**Example User filter**  
Find all traces with user IDs\.  

```
user CONTAINS ""
```

## Complex Keywords<a name="console-filters-complex"></a>

Complex keywords let you find requests based on service name, edge name, or annotation value\. For services and edges, you can specify an additional filter expression that applies to the service or edge\. For annotations, you can filter on the value of an annotation with a specific key using boolean, number or string operators\.

**Complex Keywords**
+ `service(name) {filter}` – Service with name *name*\. Optional curly braces can contain a filter expression that applies to segments created by the service\.
+ `edge(name) {filter}` – Connection between services *source* and *destination*\. Optional curly braces can contain a filter expression that applies to segments on this connection\.
+ `annotation.key` – Value of annotation with field *key*\. The value of an annotation can be a boolean, number, or string, so you can use any of those type's comparison operators\. You cannot use this keyword in combination with the `service` or `edge` keywords\.

Use the service keyword to find traces for requests that hit a certain node on your service map\.

**Example Service filter**  
Requests that included a call to `api.example.com` with a fault \(500 series error\)\.  

```
service("api.example.com") { fault }
```

You can exclude the service name to apply a filter expression to all nodes on your service map\.

**Example Service filter**  
Requests that caused a fault anywhere on your service map\.  

```
service() { fault }
```

The edge keyword applies a filter expression to a connection between two nodes\.

**Example Edge filter**  
Request where the service `api.example.com` made a call to `backend.example.com` that failed with an error\.  

```
edge("api.example.com", "backend.example.com") { error }
```

You can also use the `!` operator with service and edge keywords to exclude a service or edge from the results of another filter expression\.

**Example Service and request filter**  
Request where the URL begins with `http://api.example.com/` and contains `/v2/` but does not reach a service named `api.example.com`\.  

```
http.url BEGINSWITH "http://api.example.com/" AND http.url CONTAINS "/v2/" AND !service("api.example.com")
```

For annotations, use the comparison operators that correspond to the type of value\.

**Example Annotation with string value**  
Requests with an annotation named `gameid` with string value `"817DL6VO"`\.  

```
annotation.gameid = "817DL6VO"
```

**Example Annotation with number value**  
Requests with annotation age with numerical value greater than 29\.  

```
annotation.age > 29
```

## The ID Function<a name="console-filters-functions"></a>

When you provide a service name to the `service` or `edge` keywords, you get results for all nodes that have that name\. For more precise filtering, you can use the `id` function to specify a service type in addition to a name to distinguish between nodes with the same name\.

```
id(name: "service-name", type:"service::type")
```

You can use the `id` function in place of a service name in service and edge filters\.

```
service(id(name: "service-name", type:"service::type")) { filter }
```

```
edge(id(name: "service-one", type:"service::type"), id(name: "service-two", type:"service::type")) { filter }
```

For example, the [Scorekeep sample application](xray-scorekeep.md) includes an AWS Lambda function named `random-name`\. This creates two nodes in the service map, one for the function invocation, and one for the Lambda service\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-lambda-node.png)

The two nodes have the same name but different types\. A standard service filter will find traces for both\.

**Example Service filter**  
Requests that include an error on any service named `random-name`\.  

```
service("random-name") { error }
```

Use the `id` function to narrow the search down to errors on the function itself, excluding errors from the service\.

**Example Service filter with id function**  
Requests that include an error on a service named `random-name` with type `AWS::Lambda::Function`\.  

```
service(id(name: "random-name", type: "AWS::Lambda::Function")) { error }
```

You can also exclude the name entirely, to search for nodes by type\.

**Example Service filter with id\(\) function**  
Requests that include an error on a service with type `AWS::Lambda::Function`\.  

```
service(id(type: "AWS::Lambda::Function")) { error }
```