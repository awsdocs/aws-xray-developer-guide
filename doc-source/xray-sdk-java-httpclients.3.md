# Tracing calls to downstream HTTP web services with the X\-Ray SDK for Java<a name="xray-sdk-java-httpclients"></a>

When your application makes calls to microservices or public HTTP APIs, you can use the X\-Ray SDK for Java's version of `HttpClient` to instrument those calls and add the API to the service graph as a downstream service\.

The X\-Ray SDK for Java includes `DefaultHttpClient` and `HttpClientBuilder` classes that can be used in place of the Apache HttpComponents equivalents to instrument outgoing HTTP calls\.
+ `com.amazonaws.xray.proxies.apache.http.DefaultHttpClient` \- `org.apache.http.impl.client.DefaultHttpClient`
+ `com.amazonaws.xray.proxies.apache.http.HttpClientBuilder` \- `org.apache.http.impl.client.HttpClientBuilder`

These libraries are in the [`aws-xray-recorder-sdk-apache-http`](xray-sdk-java.md) submodule\.

You can replace your existing import statements with the X\-Ray equivalent to instrument all clients, or use the fully qualified name when you initialize a client to instrument specific clients\.

**Example HttpClientBuilder**  

```
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.util.EntityUtils;
import [com\.amazonaws\.xray\.proxies\.apache\.http\.HttpClientBuilder](https://docs.aws.amazon.com/xray-sdk-for-java/latest/javadoc/com/amazonaws/xray/proxies/apache/http/HttpClientBuilder.html);
...
  public String randomName() throws IOException {
    CloseableHttpClient httpclient = HttpClientBuilder.create().build();
    HttpGet httpGet = new HttpGet("http://names.example.com/api/");
    CloseableHttpResponse response = httpclient.execute(httpGet);
    try {
      HttpEntity entity = response.getEntity();
      InputStream inputStream = entity.getContent();
      ObjectMapper mapper = new ObjectMapper();
      Map<String, String> jsonMap = mapper.readValue(inputStream, Map.class);
      String name = jsonMap.get("name");
      EntityUtils.consume(entity);
      return name;
    } finally {
      response.close();
    }
  }
```

When you instrument a call to a downstream web api, the X\-Ray SDK for Java records a subsegment with information about the HTTP request and response\. X\-Ray uses the subsegment to generate an inferred segment for the remote API\.

**Example Subsegment for a downstream HTTP call**  

```
{
  "id": "004f72be19cddc2a",
  "start_time": 1484786387.131,
  "end_time": 1484786387.501,
  "name": "names.example.com",
  "namespace": "remote",
  "http": {
    "request": {
      "method": "GET",
      "url": "https://names.example.com/"
    },
    "response": {
      "content_length": -1,
      "status": 200
    }
  }
}
```

**Example Inferred segment for a downstream HTTP call**  

```
{
  "id": "168416dc2ea97781",
  "name": "names.example.com",
  "trace_id": "1-5880168b-fd5153bb58284b67678aa78c",
  "start_time": 1484786387.131,
  "end_time": 1484786387.501,
  "parent_id": "004f72be19cddc2a",
  "http": {
    "request": {
      "method": "GET",
      "url": "https://names.example.com/"
    },
    "response": {
      "content_length": -1,
      "status": 200
    }
  },
  "inferred": true
}
```