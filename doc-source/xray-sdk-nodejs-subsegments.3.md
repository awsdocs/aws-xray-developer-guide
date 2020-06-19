# Generating custom subsegments with the X\-Ray SDK for Node\.js<a name="xray-sdk-nodejs-subsegments"></a>

Subsegments extend a trace's [segment](xray-concepts.md#xray-concepts-segments) with details about work done in order to serve a request\. Each time you make a call with an instrumented client, the X\-Ray SDK records the information generated in a subsegment\. You can create additional subsegments to group other subsegments, to measure the performance of a section of code, or to record annotations and metadata\.

## Custom Express subsegments<a name="xray-sdk-nodejs-subsegments-express"></a>

To create a custom subsegment for a function that makes calls to downstream services, use the `captureAsyncFunc` function\.

**Example app\.js \- custom subsegments Express**  

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

**Example Subsegment with metadata**  

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

## Custom Lambda subsegments<a name="xray-sdk-nodejs-subsegments-lambda"></a>

The SDK is configured to automatically create a placeholder facade segment when the it detects it's running in Lambda\. To create a basic subsegement, which will create a single `AWS::Lambda::Function` node on the X\-Ray service map, call and repurpose the facade segment\. If you manually create a new segment with a new ID \(while sharing the trace ID, parent ID and the sampling decision\) you will be able to send a new segment\.

**Example app\.js \- manual custom subsegments**  

```
const segment = AWSXRay.getSegment(); //returns the facade segment
const subsegment = segment.addNewSubsegment('subseg');
...
subsegment.close();
//the segment is closed by the SDK automatically
```