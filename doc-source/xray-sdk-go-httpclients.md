# Tracing Calls to Downstream HTTP Web Services with the X\-Ray SDK for Go<a name="xray-sdk-go-httpclients"></a>

When your application makes calls to microservices or public HTTP APIs, you can use the `xray.Client` to instrument those calls as subsegments of your Go application, as shown in the following example, where *http\-client* is an HTTP client\.

Client creates a shallow copy of the provided http client, defaulting to `http.DefaultClient`, with roundtripper wrapped with `xray.RoundTripper`\.

**Example main\.go – HTTP client**  

```
myClient := xray.Client(http-client)
```

**Custom HTTP client**

```
request, err := httpNewRequest("PUT", myURL, bytes.NewBuffer(myPayload))
myClient := xray.Client(&http.Client{})
response, err := client.Do(request.WithContext(ctx))
```
