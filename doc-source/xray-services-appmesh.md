# Amazon EC2 and AWS App Mesh<a name="xray-services-appmesh"></a>

AWS X\-Ray integrates with [AWS App Mesh](https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html) to manage Envoy proxies for microservices\. App Mesh provides a version of Envoy that you can configure to send trace data to the X\-Ray daemon running in a container of the same task or pod\. X\-Ray supports tracing with the following App Mesh compatible services: 
+ Amazon Elastic Container Service \(Amazon ECS\)
+ Amazon Elastic Kubernetes Service \(Amazon EKS\)
+ Amazon Elastic Compute Cloud \(Amazon EC2\)

Use the following instructions to learn how to enable X\-Ray tracing through App Mesh\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/xray/latest/devguide/images/appmesh-traceContents.png)

To configure the Envoy proxy to send data to X\-Ray, set the `ENABLE_ENVOY_XRAY_TRACING` [environment variable](https://docs.aws.amazon.com/app-mesh/latest/userguide/envoy.html#envoy-config) in its container definition\.

**Example Envoy container definition for Amazon ECS**  

```
{
      "name": "envoy",
      "image": "840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.12.3.0-prod",
      "essential": true,
      "environment": [
        {
          "name": "APPMESH_VIRTUAL_NODE_NAME",
          "value": "mesh/myMesh/virtualNode/myNode"
        },
        {
          "name": "ENABLE_ENVOY_XRAY_TRACING",
          "value": "1"
        }
      ],
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -s http://localhost:9901/server_info | cut -d' ' -f3 | grep -q live"
        ],
        "startPeriod": 10,
        "interval": 5,
        "timeout": 2,
        "retries": 3
      }
```

**Note**  
To learn more about available Envoy region addresses, see [Envoy image](https://docs.aws.amazon.com/app-mesh/latest/userguide/envoy.html) in the AWS App Mesh User Guide\.

For details on running the X\-Ray daemon in a container, see [Running the X\-Ray daemon on Amazon ECS](xray-daemon-ecs.md)\. For a sample application that includes a service mesh, microservice, Envoy proxy, and X\-Ray daemon, deploy the `colorapp` sample in the [App Mesh Examples GitHub repository](https://github.com/aws/aws-app-mesh-examples/tree/master/examples)\.

**Learn More**
+ [Getting Started with AWS App Mesh](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting_started.html)
+ [Getting Started with AWS App Mesh and Amazon ECS](https://docs.aws.amazon.com/app-mesh/latest/userguide/mesh-getting-started-ecs.html)