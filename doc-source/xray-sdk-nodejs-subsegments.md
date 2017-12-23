# Generating Custom Subsegments with the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-subsegments"></a>

A segment is a JSON document that records the work that your application does to serve a single request\. The X\-Ray SDK for Node\.js middleware creates segments for HTTP requests and adds details about the request and response, including information from headers in the request, the time that the request was received, and the time that the response was sent\.

Further instrumentation generates *subsegments*\. Instrumented AWS SDK clients and HTTP clients add subsegments to the segment document with details of downstream calls made by the application\.

You can create subsegments manually to instrument functions and organize other subsegments into groups\. For example, you can create a custom subsegment for a function that makes calls to downstream services with the `captureAsyncFunc` function\.

**Example app\.js \- Custom Subsegments**  

```
var AWSXRay = require('aws-xray-sdk');

app.use(AWSXRay.express.openSegment('MyApp'));

app.get('/', function (req, res) {
  var host = 'api.example.com';

  AWSXRay.captureAsyncFunc('send', function(subsegment) {
    sendRequest(host, function() {
      console.log('rendering!');
      res.render('index');
      subsegment.close();
    });
  });
});

app.use(AWSXRay.express.closeSegment());

function sendRequest(host, cb) {
  var options = {
    host: host,
    path: '/',
  };

  var callback = function(response) {
    var str = '';

    response.on('data', function (chunk) {
      str += chunk;
    });

    response.on('end', function () {
      cb();
    });
  }

  http.request(options, callback).end();
};
```

In this example, the application creates a custom subsegment named `send` for calls to the `sendRequest` function\. `captureAsyncFunc` passes a subsegment that you must close within the callback function when the asynchronous calls that it makes are complete\.

For synchronous functions, you can use the `captureFunc` function, which closes the subsegment automatically as soon as the function block finishes executing\.

When you create a subsegment within a segment or another subsegment, the X\-Ray SDK for Node\.js generates an ID for it and records the start time and end time\.

**Example Subsegment with Metadata**  

```
"subsegments": [{
  "id": "6f1605cd8a07cb70",
  "start_time": 1.480305974194E9,
  "end_time": 1.4803059742E9,
  "name": "Custom subsegment for UserModel.saveUser function",
  "metadata": {
    "debug": {
      "test": "Metadata string from UserModel.saveUser"
    }
  },
```