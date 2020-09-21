# Using filter expressions to search for traces in the console<a name="xray-console-filters"></a>

When you choose a time period of traces to view in the X\-Ray console, you might get more results than the console can display\. In the upper\-right corner, the console shows the number of traces that it scanned and whether there are more traces available\. 

You can narrow the results to just the traces that you want to find by using a *filter expression*\.

**Topics**
+ [Filter expression details](#xray-console-filters-details)
+ [Using filter expressions with groups](#groups)
+ [Filter expression syntax](#console-filters-syntax)
+ [Boolean keywords](#console-filters-boolean)
+ [Number keywords](#console-filters-number)
+ [String keywords](#console-filters-string)
+ [Complex keywords](#console-filters-complex)
+ [id function](#console-filters-functions)

## Filter expression details<a name="xray-console-filters-details"></a>

When you [choose a node in the service map](xray-console.md#xray-console-servicemap), the console constructs a filter expression based on the service name of the node, and the types of error present based on your selection\. To find traces that show performance issues or that relate to specific requests, you can adjust the expression that the console provides or create your own\. If you add annotations with the X\-Ray SDK, you can also filter based on the presence of an annotation key or the value of a key\.

**Note**  
If you choose a relative time range in the service map and choose a node, the console converts the time range to an absolute start and end time\. To ensure that the traces for the node appear in the search results, and avoid scanning times when the node wasn't active, the time range only includes times when the node sent traces\. To search relative to the current time, you can switch back to a relative time range in the traces page and scan again\.

If there are still more results available than the console can show, the console shows you how many traces matched and the number of traces scanned\. The percentage shown is the percentage of the selected time frame that was scanned\. To ensure that you see all matching traces represented in the results, narrow your filter expression further, or choose a shorter time frame\.

To get the freshest results first, the console starts scanning at the end of the time range and works backward\. If there are a large number of traces, but few results, the console splits the time range into chunks and scans them in parallel\. The progress bar shows the parts of the time range that have been scanned\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/console-tracescan-parallel.png)

## Using filter expressions with groups<a name="groups"></a>

Groups are a collection of traces that are defined by a filter expression\. You can use groups to generate additional service graphs and supply Amazon CloudWatch metrics\. 

Groups are identified by their name or an Amazon Resource Name \(ARN\), and contain a filter expression\. The service compares incoming traces to the expression and stores them accordingly\.

You can create and modify groups by using the dropdown menu to the left of the filter expression search bar\.

**Note**  
If the service encounters an error in qualifying a group, that group is no longer included in processing incoming traces and an error metric is recorded\.

For more information about groups, see [Configuring groups in the X\-Ray console](xray-console-groups.md)\.

## Filter expression syntax<a name="console-filters-syntax"></a>

Filter expressions can contain a *keyword*, a unary or binary *operator*, and a *value* for comparison\.

```
keyword operator value
```

Different operators are available for different types of keyword\. For example, `responsetime` is a number keyword and can be compared with operators related to numbers\.

**Example – requests where response time was greater than 5 seconds**  

```
responsetime > 5
```

You can combine multiple expressions in a compound expression by using the `AND` or `OR` operators\.

**Example – requests where the total duration was 5–8 seconds**  

```
duration >= 5 AND duration <= 8
```

Simple keywords and operators find issues only at the trace level\. If an error occurs downstream, but is handled by your application and not returned to the user, a search for `error` will not find it\.

To find traces with downstream issues, you can use the [complex keywords](#console-filters-complex) `service()` and `edge()`\. These keywords let you apply a filter expression to all downstream nodes, a single downstream node, or an edge between two nodes\. For more granularity, you can filter services and edges by type with [the id\(\) function](#console-filters-functions)\.

## Boolean keywords<a name="console-filters-boolean"></a>

Boolean keyword values are either true or false\. Use these keywords to find traces that resulted in errors\.

**Boolean keywords**
+ `ok` – Response status code was 2XX Success\.
+ `error` – Response status code was 4XX Client Error\.
+ `throttle` – Response status code was 429 Too Many Requests\.
+ `fault` – Response status code was 5XX Server Error\.
+ `partial` – Request has incomplete segments\.
+ `inferred` – Request has inferred segments\.
+ `first` – Element is the first of an enumerated list\.
+ `last` – Element is the last of an enumerated list\.
+ `remote` – Root cause entity is remote\.
+ `root` – Service is the entry point or root segment of a trace\.

Boolean operators find segments where the specified key is `true` or `false`\.

**Boolean operators**
+ none – The expression is true if the keyword is true\.
+ `!` – The expression is true if the keyword is false\.
+ `=`,`!=` – Compare the value of the keyword to the string `true` or `false`\. These operators act the same as the other operators but are more explicit\.

**Example – response status is 2XX OK**  

```
ok
```

**Example – response status is not 2XX OK**  

```
!ok
```

**Example – response status is not 2XX OK**  

```
ok = false
```

**Example – last enumerated fault trace has error name "deserialize"**  

```
rootcause.fault.entity { last and name = "deserialize" }
```

**Example – requests with remote segments where coverage is greater than 0\.7 and the service name is "traces"**  

```
rootcause.responsetime.entity { remote and coverage > 0.7 and name = "traces" }
```

**Example – requests with inferred segments where the service type is "AWS:DynamoDB"**  

```
rootcause.fault.service { inferred and name = traces and type = "AWS::DynamoDB" }
```

**Example – requests that have a segment with the name "data\-plane" as the root**  

```
service("data-plane") {root = true and fault = true}
```

## Number keywords<a name="console-filters-number"></a>

Use number keywords to search for requests with a specific response time, duration, or response status\.

**Number keywords**
+ `responsetime` – Time that the server took to send a response\.
+ `duration` – Total request duration, including all downstream calls\.
+ `http.status` – Response status code\.
+ `index` – Position of an element in an enumerated list\.
+ `coverage` – Decimal percentage of entity response time over root segment response time\. Applicable only for response time root cause entities\.

**Number operators**

Number keywords use standard equality and comparison operators\.
+ `=`,`!=` – The keyword is equal to or not equal to a number value\.
+ `<`,`<=`, `>`,`>=` – The keyword is less than or greater than a number value\.

**Example – response status is not 200 OK**  

```
http.status != 200
```

**Example – request where the total duration was 5–8 seconds**  

```
duration >= 5 AND duration <= 8
```

**Example – requests that completed successfully in less than 3 seconds, including all downstream calls**  

```
ok !partial duration <3
```

**Example – enumerated list entity that has an index greater than 5**  

```
rootcause.fault.service { index > 5 }
```

**Example – requests where the last entity that has coverage greater than 0\.8**  

```
rootcause.responsetime.entity { last and coverage > 0.8 }
```

## String keywords<a name="console-filters-string"></a>

Use string keywords to find traces with specific text in the request headers, or specific user IDs\.

**String keywords**
+ `http.url` – Request URL\.
+ `http.method` – Request method\.
+ `http.useragent` – Request user agent string\.
+ `http.clientip` – Requestor's IP address\.
+ `user` – Value of the user field on any segment in the trace\.
+ `name` – The name of a service or exception\.
+ `type` – Service type\.
+ `message` – Exception message\.
+ `availabilityzone` – Value of the availabilityzone field on any segment in the trace\.
+ `instance.id` – Value of the instance ID field on any segment in the trace\.
+ `resource.arn` – Value of the resource ARN field on any segment in the trace\.

String operators find values that are equal to or contain specific text\. Values must always be specified in quotation marks\. 

**String operators**
+ `=`,`!=` – The keyword is equal to or not equal to a number value\.
+ `CONTAINS` – The keyword contains a specific string\.
+ `BEGINSWITH` , `ENDSWITH` – The keyword begins or ends with a specific string\.

**Example – http\.url filter**  

```
http.url CONTAINS "/api/game/"
```

To test if a field exists on a trace, regardless of its value, check to see if it contains the empty string\.

**Example – user filter**  
Find all traces with user IDs\.  

```
user CONTAINS ""
```

**Example – select traces with a fault root cause that includes a service named "Auth"**  

```
rootcause.fault.service { name = "Auth" }
```

**Example – select traces with a response time root cause whose last service has a type of DynamoDB**  

```
rootcause.responsetime.service { last and type = "AWS::DynamoDB" }
```

**Example – select traces with a fault root cause whose last exception has the message "access denied for account\_id: 1234567890"**  

```
rootcause.fault.exception { last and message = "Access Denied for account_id: 1234567890" 
```

## Complex keywords<a name="console-filters-complex"></a>

Use complex keywords to find requests based on service name, edge name, or annotation value\. For services and edges, you can specify an additional filter expression that applies to the service or edge\. For annotations, you can filter on the value of an annotation with a specific key using Boolean, number, or string operators\.

**Complex keywords**
+ `annotation.key` – Value of an annotation with field *key*\. The value of an annotation can be a Boolean, number, or string, so you can use any of the comparison operators of those types\. You can't use this keyword in combination with the `service` or `edge` keyword\.
+ `edge(source, destination) {filter}` – Connection between services *source* and *destination*\. Optional curly braces can contain a filter expression that applies to segments on this connection\.
+ `group.name / group.arn` – The value of a group's filter expression, referenced by group name or group ARN\.
+ `json` – JSON root cause object\. See [Getting data from AWS X\-Ray](xray-api-gettingdata.md) for steps to create JSON entities programmatically\.
+ `service(name) {filter}` – Service with name *name*\. Optional curly braces can contain a filter expression that applies to segments created by the service\.

Use the service keyword to find traces for requests that hit a certain node on your service map\.

Complex keyword operators find segments where the specified key has been set, or not set\.

**Complex keyword operators**
+ none – The expression is true if the keyword is set\. If the keyword is of boolean type, it will evaluate to the boolean value\.
+ `!` – The expression is true if the keyword is not set\. If the keyword is of boolean type, it will evaluate to the boolean value\.
+ `=`,`!=` – Compare the value of the keyword\.
+ `edge(source, destination) {filter}` – Connection between services *source* and *destination*\. Optional curly braces can contain a filter expression that applies to segments on this connection\.
+ `annotation.key` – Value of an annotation with field *key*\. The value of an annotation can be a Boolean, number, or string, so you can use any of the comparison operators of those types\. You can't use this keyword in combination with the `service` or `edge` keyword\.
+ `json` – JSON root cause object\. See [Getting data from AWS X\-Ray](xray-api-gettingdata.md) for steps to create JSON entities programmatically\.

Use the service keyword to find traces for requests that hit a certain node on your service map\.

**Example – Service filter**  
Requests that included a call to `api.example.com` with a fault \(500 series error\)\.  

```
service("api.example.com") { fault }
```

You can exclude the service name to apply a filter expression to all nodes on your service map\.

**Example – service filter**  
Requests that caused a fault anywhere on your service map\.  

```
service() { fault }
```

The edge keyword applies a filter expression to a connection between two nodes\.

**Example – edge filter**  
Request where the service `api.example.com` made a call to `backend.example.com` that failed with an error\.  

```
edge("api.example.com", "backend.example.com") { error }
```

You can also use the `!` operator with service and edge keywords to exclude a service or edge from the results of another filter expression\.

**Example – service and request filter**  
Request where the URL begins with `http://api.example.com/` and contains `/v2/` but does not reach a service named `api.example.com`\.  

```
http.url BEGINSWITH "http://api.example.com/" AND http.url CONTAINS "/v2/" AND !service("api.example.com")
```

**Example – service and response time filter**  
Find traces where `http url` is set and response time is greater than 2 seconds\.  

```
http.url AND responseTime > 2
```

For annotations, you can call all traces where `annotation.key` is set, or use the comparison operators that correspond to the type of value\.

**Example – annotation with string value**  
Requests with an annotation named `gameid` with string value `"817DL6VO"`\.  

```
annotation.gameid = "817DL6VO"
```

**Example – annotation is set**  
Requests with an annotation named `age` set\.  

```
annotation.age
```

**Example – annotation is not set**  
Requests without an annotation named `age` set\.  

```
!annotation.age
```

**Example – annotation with number value**  
Requests with annotation age with numerical value greater than 29\.  

```
annotation.age > 29
```

**Example – group with user**  
Requests where traces meet the `high_response_time` group filter \(e\.g\. `responseTime > 3`\), and the user is named Alice\.  

```
group.name = "high_response_time" AND user = "alice"
```

**Example – JSON with root cause entity**  
Requests with matching root cause entities\.  

```
rootcause.json = #[{ "Services": [ { "Name": "GetWeatherData", "EntityPath": [{ "Name": "GetWeatherData" }, { "Name": "get_temperature" } ] }, { "Name": "GetTemperature", "EntityPath": [ { "Name": "GetTemperature" } ] } ] }]
```

## id function<a name="console-filters-functions"></a>

When you provide a service name to the `service` or `edge` keyword, you get results for all nodes that have that name\. For more precise filtering, you can use the `id` function to specify a service type in addition to a name to distinguish between nodes with the same name\.

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

**Example – service filter**  
Requests that include an error on any service named `random-name`\.  

```
service("random-name") { error }
```

Use the `id` function to narrow the search to errors on the function itself, excluding errors from the service\.

**Example – service filter with id function**  
Requests that include an error on a service named `random-name` with type `AWS::Lambda::Function`\.  

```
service(id(name: "random-name", type: "AWS::Lambda::Function")) { error }
```

To search for nodes by type, you can also exclude the name entirely\.

**Example – service filter with id function**  
Requests that include an error on a service with type `AWS::Lambda::Function`\.  

```
service(id(type: "AWS::Lambda::Function")) { error }
```