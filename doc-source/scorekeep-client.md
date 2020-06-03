# Instrumenting a web app client<a name="scorekeep-client"></a>

In the [https://github.com/awslabs/eb-java-scorekeep/tree/xray-cognito](https://github.com/awslabs/eb-java-scorekeep/tree/xray-cognito) branch, Scorekeep uses Amazon Cognito to enable users to create an account and sign in with it to retrieve their user information from an Amazon Cognito user pool\. When a user signs in, Scorekeep uses an Amazon Cognito identity pool to get temporary AWS credentials for use with the AWS SDK for JavaScript\.

The identity pool is configured to let signed\-in users write trace data to AWS X\-Ray\. The web app uses these credentials to record the signed\-in user's ID, the browser path, and the client's view of calls to the Scorekeep API\.

Most of the work is done in a service class named `xray`\. This service class provides methods for generating the required identifiers, creating in\-progress segments, finalizing segments, and sending segment documents to the X\-Ray API\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray-cognito/public/app/xray.js](https://github.com/awslabs/eb-java-scorekeep/tree/xray-cognito/public/app/xray.js) – Record and upload segments**  

```
...
  service.beginSegment = function() {
    var segment = {};
    var traceId = '1-' + service.getHexTime() + '-' + service.getHexId(24);

    var id = service.getHexId(16);
    var startTime = service.getEpochTime();

    segment.trace_id = traceId;
    segment.id = id;
    segment.start_time = startTime;
    segment.name = 'Scorekeep-client';
    segment.in_progress = true;
    segment.user =  sessionStorage['userid'];
    segment.http = {
      request: {
        url: window.location.href
      }
    };

    var documents = [];
    documents[0] = JSON.stringify(segment);
    service.putDocuments(documents);
    return segment;
  }

  service.endSegment = function(segment) {
    var endTime = service.getEpochTime();
    segment.end_time = endTime;
    segment.in_progress = false;
    var documents = [];
    documents[0] = JSON.stringify(segment);
    service.putDocuments(documents);
  }

  service.putDocuments = function(documents) {
    var xray = new AWS.XRay();
    var params = {
      TraceSegmentDocuments: documents
    };
    xray.putTraceSegments(params, function(err, data) {
      if (err) {
        console.log(err, err.stack);
      } else {
        console.log(data);
      }
    })
  }
```

These methods are called in header and `transformResponse` functions in the resource services that the web app uses to call the Scorekeep API\. To include the client segment in the same trace as the segment that the API generates, the web app must include the trace ID and segment ID in a [tracing header](xray-concepts.md#xray-concepts-tracingheader) \(`X-Amzn-Trace-Id`\) that the X\-Ray SDK can read\. When the instrumented Java application receives a request with this header, the X\-Ray SDK for Java uses the same trace ID and makes the segment from the web app client the parent of its segment\. 

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/public/app/services.js](https://github.com/awslabs/eb-java-scorekeep/tree/xray/public/app/services.js) – Recording segments for angular resource calls and writing tracing headers**  

```
var module = angular.module('scorekeep');
module.factory('SessionService', function($resource, api, XRay) {
  return $resource(api + 'session/:id', { id: '@_id' }, {
    segment: {},
    get: {
      method: 'GET',
      headers: {
        'X-Amzn-Trace-Id': function(config) {
          segment = XRay.beginSegment();
          return XRay.getTraceHeader(segment);
        }
      },
      transformResponse: function(data) {
        XRay.endSegment(segment);
        return angular.fromJson(data);
      },
    },
...
```

The resulting service map includes a node for the web app client\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-client.png)

Traces that include segments from the web app show the URL that the user sees in the browser \(paths starting with `/#/`\)\. Without client instrumentation, you only get the URL of the API resource that the web app calls \(paths starting with `/api/`\)\.

![\[\]](http://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-traces-client.png)