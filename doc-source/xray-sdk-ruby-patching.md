# Patching libraries to instrument downstream calls<a name="xray-sdk-ruby-patching"></a>

To instrument downstream calls, use the X\-Ray SDK for Ruby to patch the libraries that your application uses\. The X\-Ray SDK for Ruby can patch the following libraries\.

**Supported Libraries**
+ `[net/http](https://ruby-doc.org/stdlib-2.4.2/libdoc/net/http/rdoc/Net/HTTP.html)` – Instrument HTTP clients\.
+ `[aws\-sdk](https://aws.amazon.com/sdk-for-ruby)` – Instrument AWS SDK for Ruby clients\.

When you use a patched library, the X\-Ray SDK for Ruby creates a subsegment for the call and records information from the request and response\. A segment must be available for the SDK to create the subsegment, either from the SDK middleware or a call to `XRay.recorder.begin_segment`\.

To patch libraries, specify them in the configuration object that you pass to the X\-Ray recorder\.

**Example main\.rb – Patch libraries**  

```
require 'aws-xray-sdk'

config = {
  name: 'my app',
  patch: %I[net_http aws_sdk]
}

XRay.recorder.configure(config)
```