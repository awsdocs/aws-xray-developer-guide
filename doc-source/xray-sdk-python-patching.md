# Patching libraries to instrument downstream calls<a name="xray-sdk-python-patching"></a>

To instrument downstream calls, use the X\-Ray SDK for Python to patch the libraries that your application uses\. The X\-Ray SDK for Python can patch the following libraries\.

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

When you use a patched library, the X\-Ray SDK for Python creates a subsegment for the call and records information from the request and response\. A segment must be available for the SDK to create the subsegment, either from the SDK middleware or from AWS Lambda\.

**Note**  
If you use SQLAlchemy ORM, you can instrument your SQL queries by importing the SDK's version of SQLAlchemy's session and query classes\. See [Use SQLAlchemy ORM](https://github.com/aws/aws-xray-sdk-python/blob/master/README.md#use-sqlalchemy-orm) for instructions\.

To patch all available libraries, use the `patch_all` function in `aws_xray_sdk.core`\. Some libraries, such as `httplib` and `urllib`, may need to enable double patching by calling `patch_all(double_patch=True)`\.

**Example main\.py – Patch all supported libraries**  

```
import boto3
import botocore
import requests
import sqlite3

from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()
```

To patch a single library, call `patch` with a tuple of the library name\. In order to achieve this, you will need to provide a single element list\.

**Example main\.py – Patch specific libraries**  

```
import boto3
import botocore
import requests
import mysql-connector-python

from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch

libraries = (['botocore'])
patch(libraries)
```

**Note**  
In some cases, the key that you use to patch a library does not match the library name\. Some keys serve as aliases for one or more libraries\.  
`httplib` – `[httplib](https://docs.python.org/2/library/httplib.html)` and [https://docs.python.org/3/library/http.client.html](https://docs.python.org/3/library/http.client.html)
`mysql` – `[mysql\-connector\-python](https://pypi.python.org/pypi/mysql-connector-python)`

## Tracing context for asynchronous work<a name="xray-sdk-python-patching-async"></a>

For `asyncio` integrated libraries, or to [create subsegments for asynchronous functions](xray-sdk-python-subsegments.md), you must also configure the X\-Ray SDK for Python with an async context\. Import the `AsyncContext` class and pass an instance of it to the X\-Ray recorder\.

**Note**  
Web framework support libraries, such as AIOHTTP, are not handled through the `aws_xray_sdk.core.patcher` module\. They will not appear in the `patcher` catalog of supported libraries\.

**Example main\.py – Patch aioboto3**  

```
import asyncio
import aioboto3
import requests

from aws_xray_sdk.core.async_context import AsyncContext
from aws_xray_sdk.core import xray_recorder
xray_recorder.configure(service='my_service', context=AsyncContext())
from aws_xray_sdk.core import patch

libraries = ('aioboto3')
patch(libraries)
```