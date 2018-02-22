# Patching Libraries to Instrument Downstream Calls<a name="xray-sdk-python-patching"></a>

To instrument downstream calls, use the X\-Ray SDK for Python to patch the libraries that your application uses\. The X\-Ray SDK for Python can patch the following libraries\.

**Supported Libraries**

+ `[botocore](https://pypi.python.org/pypi/botocore)` and `[boto3](https://pypi.python.org/pypi/boto3)` – Instrument AWS SDK for Python \(Boto\) clients\.

+ `[aiobotocore](https://pypi.python.org/pypi/aiobotocore)` and `[aioboto3](https://pypi.python.org/pypi/aioboto3)` – Instrument [asyncio](https://docs.python.org/3/library/asyncio.html)\-integrated versions of SDK for Python clients\.

+ `[requests](https://pypi.python.org/pypi/requests)`, `[aiohttp](https://pypi.python.org/pypi/aiohttp)` – Instrument HTTP clients\.

+ `[sqlite3](https://docs.python.org/3/library/sqlite3.html)` – Instrument SQLite clients\.

+ `[mysql\-connector\-python](https://pypi.python.org/pypi/mysql-connector-python)` – Instrument MySQL clients\.

When you use a patched library, the X\-Ray SDK for Python creates a subsegment for the call and records information from the request and response\. A segment must be available for the SDK to create the subsegment, either from the SDK middleware or from AWS Lambda\.

To patch all available libraries, use the `patch_all` function in `aws_xray_sdk.core`\.

**Example main\.py – patch all supported libraries**  

```
import boto3
import botocore
import requests
import sqlite3

from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()
```

To patch individual libraries, call `patch` with a tuple of library names\.

**Example main\.py – patch specific libraries**  

```
import boto3
import botocore
import requests
import sqlite3

from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch

libraries = ('botocore', 'requests')
patch(libraries)
```

For `asyncio` integrated libraries, or to [create subsegments for asynchronous functions](xray-sdk-python-subsegments.md), you must also configure the X\-Ray SDK for Python with an async context\.

**Example main\.py – patch aioboto3**  

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