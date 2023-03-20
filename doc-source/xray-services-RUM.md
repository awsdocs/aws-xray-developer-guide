# CloudWatch RUM and AWS X\-Ray<a name="xray-services-RUM"></a>

With Amazon CloudWatch RUM, you can perform real user monitoring to collect and view client\-side data about your web application performance from actual user sessions in near\-real time\. With AWS X\-Ray and CloudWatch RUM, you can analyze and debug the request path starting from end users of your application through downstream AWS managed services\. This helps you identify latency trends and errors that impact your end users\. 

After you turn on X\-Ray tracing of user sessions, CloudWatch RUM adds an X\-Ray trace header to allowed HTTP requests, and records an X\-Ray segment for allowed HTTP requests\. You can then see traces and segments from these user sessions in the X\-Ray and CloudWatch consoles, including the X\-Ray service map\. 

**Note**  
CloudWatch RUM doesn't integrate with X\-Ray sampling rules\. Instead, choose a sampling percentage when you set up your application to use CloudWatch RUM\. Traces sent from CloudWatch RUM might incur additional costs\. For more information, see [AWS X\-Ray pricing](https://aws.amazon.com/xray/pricing/)\. 

By default, client\-side traces sent from CloudWatch RUM aren't connected to server\-side traces\. To connect client\-side traces with server\-side traces, configure the CloudWatch RUM web client to add an X\-Ray trace header to these HTTP requests\. 

**Warning**  
Configuring the CloudWatch RUM web client to add an X\-Ray trace header to HTTP requests can cause cross\-origin resource sharing \(CORS\) to fail. To avoid this, add the `X-Amzn-Trace-Id` HTTP header to the list of allowed headers on your downstream service's CORS configuration. If you are using API Gateway as your downstream, see [Enabling CORS for a REST API resource](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html). We strongly recommend that you test your application before adding a client\-side X\-Ray trace header in a production environment\. For more information, see the [ CloudWatch RUM web client documentation](https://github.com/aws-observability/aws-rum-web/blob/main/docs/cdn_installation.md#http)\. 


For more information about real user monitoring in CloudWatch, see [Use CloudWatch RUM](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-RUM.html)\. To set up your application to use CloudWatch RUM, including tracing user sessions with X\-Ray, see [Set up an application to use CloudWatch RUM](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-RUM-get-started.html)\. 