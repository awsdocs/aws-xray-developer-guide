# Tracing calls to downstream HTTP web services with the X\-Ray SDK for Go<a name="xray-sdk-go-httpclients"></a>

When your application makes calls to microservices or public HTTP APIs, you can use the `xray.Client` to instrument those calls as subsegments of your Go application, as shown in the following example, where *http\-client* is an HTTP client\.

The client creates a shallow copy of the provided HTTP client, defaulting to `http.DefaultClient`, with roundtripper wrapped with `xray.RoundTripper`\.

**Example main\.go – HTTP client**  

```
myClient := xray.Client(http-client)
```

**Example main\.go – Trace downstream HTTP call with ctxhttp library**

The following example instruments the outgoing HTTP call with the ctxhttp library using `xray.Client`. `ctx` can be passed from the upstream call. This ensures that the existing segment context is used. For example, X-Ray does not allow a new segment to be created within a Lambda function, so the existing Lambda segment context should be used.

```
resp, err := ctxhttp.Get(ctx, xray.Client(nil), url)
```
