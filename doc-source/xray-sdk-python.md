# AWS X\-Ray SDK for Python<a name="xray-sdk-python"></a>

The X\-Ray SDK for Python is a library for Python web applications that provides classes and methods for generating and sending trace data to the X\-Ray daemon\. Trace data includes information about incoming HTTP requests served by the application, and calls that the application makes to downstream services using the AWS SDK, HTTP clients, or an SQL database connector\. You can also create segments manually and add debug information in annotations and metadata\.

You can download the SDK with `pip`\.

```
$ pip install aws-xray-sdk
```

**Note**  
The X\-Ray SDK for Python is an open source project\. You can follow the project and submit issues and pull requests on GitHub: [github\.com/aws/aws\-xray\-sdk\-python](https://github.com/aws/aws-xray-sdk-python)

If you use Django or Flask, start by [adding the SDK middleware to your application](xray-sdk-python-middleware.md) to trace incoming requests\. The middleware creates a [segment](xray-concepts.md#xray-concepts-segments) for each traced request, and completes the segment when the response is sent\. While the segment is open, you can use the SDK client's methods to add information to the segment and create subsegments to trace downstream calls\. The SDK also automatically records exceptions that your application throws while the segment is open\. For other applications, you can [create segments manually](xray-sdk-python-middleware.md#xray-sdk-python-middleware-manual)\.

For Lambda functions called by an instrumented application or service, Lambda reads the [tracing header](xray-concepts.md#xray-concepts-tracingheader) and traces sampled requests automatically\. For other functions, you can [configure Lambda](xray-services-lambda.md) to sample and trace incoming requests\. In either case, Lambda creates the segment and provides it to the X\-Ray SDK\.

**Note**  
On Lambda, the X\-Ray SDK is optional\. If you don't use it in your function, your service map will still include a node for the Lambda service, and one for each Lambda function\. By adding the SDK, you can instrument your function code to add subsegments to the function segment recorded by Lambda\. See [AWS Lambda and AWS X\-Ray](xray-services-lambda.md) for more information\.

See [Worker](scorekeep-lambda.md#scorekeep-lambda-worker) for a example Python function instrumented in Lambda\.

Next, use the X\-Ray SDK for Python to instrument downstream calls by [patching your application's libraries](xray-sdk-python-patching.md)\. The SDK supports the following libraries\.

**Supported Libraries**
+ `[botocore](https://pypi.python.org/pypi/botocore)`, `[boto3](https://pypi.python.org/pypi/boto3)` – Instrument AWS SDK for Python \(Boto\) clients\.
+ `[pynamodb](https://pypi.python.org/pypi/pynamodb/)` – Instrument PynamoDB's version of the Amazon DynamoDB client\.
+ `[aiobotocore](https://pypi.python.org/pypi/aiobotocore)`, `[aioboto3](https://pypi.python.org/pypi/aioboto3)` – Instrument [asyncio](https://docs.python.org/3/library/asyncio.html)\-integrated versions of SDK for Python clients\.
+ `[requests](https://pypi.python.org/pypi/requests)`, `[aiohttp](https://pypi.python.org/pypi/aiohttp)` – Instrument high\-level HTTP clients\.
+ `[httplib](https://docs.python.org/2/library/httplib.html)`, [https://docs.python.org/3/library/http.client.html](https://docs.python.org/3/library/http.client.html) – Instrument low\-level HTTP clients and the higher level libraries that use them\.
+ `[sqlite3](https://docs.python.org/3/library/sqlite3.html)` – Instrument SQLite clients\.
+ `[mysql\-connector\-python](https://pypi.python.org/pypi/mysql-connector-python)` – Instrument MySQL clients\.
+ `[pg8000](https://pypi.org/project/pg8000/)` – Instrument Pure\-Python PostgreSQL interface\.
+ `[psycopg2](https://pypi.org/project/psycopg2/)` – Instrument PostgreSQL database adapter\.
+ `[pymongo](https://pypi.org/project/pymongo/)` – Instrument MongoDB clients\.
+ `[pymysql](https://pypi.org/project/PyMySQL/)` – Instrument PyMySQL based clients for MySQL and MariaDB\.

Whenever your application makes calls to AWS, an SQL database, or other HTTP services, the SDK records information about the call in a subsegment\. AWS services and the resources that you access within the services appear as downstream nodes on the service map to help you identify errors and throttling issues on individual connections\.

Once you get going with the SDK, customize its behavior by [configuring the recorder and middleware](xray-sdk-python-configuration.md)\. You can add plugins to record data about the compute resources running your application, customize sampling behavior by defining sampling rules, and set the log level to see more or less information from the SDK in your application logs\.

Record additional information about requests and the work that your application does in [annotations and metadata](xray-sdk-python-segment.md)\. Annotations are simple key\-value pairs that are indexed for use with [filter expressions](xray-console-filters.md), so that you can search for traces that contain specific data\. Metadata entries are less restrictive and can record entire objects and arrays — anything that can be serialized into JSON\.

**Annotations and Metadata**  
Annotations and metadata are arbitrary text that you add to segments with the X\-Ray SDK\. Annotations are indexed for use with filter expressions\. Metadata are not indexed, but can be viewed in the raw segment with the X\-Ray console or API\. Anyone that you grant read access to X\-Ray can view this data\.

When you have a lot of instrumented clients in your code, a single request segment can contain a large number of subsegments, one for each call made with an instrumented client\. You can organize and group subsegments by wrapping client calls in [custom subsegments](xray-sdk-python-subsegments.md)\. You can create a custom subsegment for an entire function or any section of code\. You can then you can record metadata and annotations on the subsegment instead of writing everything on the parent segment\.

For reference documentation for the SDK's classes and methods, see the [AWS X\-Ray SDK for Python API Reference](https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference)\.

## Requirements<a name="xray-sdk-python-requirements"></a>

The X\-Ray SDK for Python supports the following language and library versions\.
+ **Python** – 2\.7, 3\.4, and newer
+ **Django** – 1\.10 and newer
+ **Flask** – 0\.10 and newer
+ **aiohttp** – 2\.3\.0 and newer
+ **AWS SDK for Python \(Boto\)** – 1\.4\.0 and newer
+ **botocore** – 1\.5\.0 and newer
+ **enum** – 0\.4\.7 and newer, for Python versions 3\.4\.0 and older
+ **jsonpickle** – 1\.0\.0 and newer
+ **setuptools** – 40\.6\.3 and newer
+ **wrapt** – 1\.11\.0 and newer

## Dependency management<a name="xray-sdk-python-dependencies"></a>

The X\-Ray SDK for Python is available from `pip`\.
+ **Package** – `aws-xray-sdk`

Add the SDK as a dependency in your `requirements.txt` file\.

**Example requirements\.txt**  

```
aws-xray-sdk==2.4.2
boto3==1.4.4
botocore==1.5.55
Django==1.11.3
```

If you use Elastic Beanstalk to deploy your application, Elastic Beanstalk installs all of the packages in `requirements.txt` automatically\.