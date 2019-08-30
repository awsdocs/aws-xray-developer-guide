# Running the X\-Ray Daemon on Amazon ECS<a name="xray-daemon-ecs"></a>

In Amazon ECS, create a Docker image that runs the X\-Ray daemon, upload it to a Docker image repository, and then deploy it to your Amazon ECS cluster\. You can use port mappings and network mode settings in your task definition file to allow your application to communicate with the daemon container\.

## Using the Official Docker Image<a name="xray-daemon-ecs-image"></a>

X\-Ray provides a Docker container image that you can deploy alongside your application\.

```
$ docker pull amazon/aws-xray-daemon
```

**Example Task definition**  

```
    {
      "name": "xray-daemon",
      "image": "amazon/aws-xray-daemon",
      "cpu": 32,
      "memoryReservation": 256,
      "portMappings" : [
          {
              "hostPort": 0,
              "containerPort": 2000,
              "protocol": "udp"
          }
       ],
    }
```

## Create and Build a Docker Image<a name="xray-daemon-ecs-build"></a>

For custom configuration, you may need to define your own Docker image\.

**Note**  
The Scorekeep sample application shows how to use the X\-Ray daemon on Amazon ECS\. See [Instrumenting Amazon ECS Applications](scorekeep-ecs.md) for details\.

Add managed policies to your task role to grant the daemon permission to upload trace data to X\-Ray\. For more information, see [Giving the Daemon Permission to Send Data to X\-Ray](xray-daemon.md#xray-daemon-permissions)\.

Use one of the following Dockerfiles to create an image that runs the daemon\.

**Example Dockerfile – Amazon Linux**  

```
FROM amazonlinux
RUN yum install -y unzip
RUN curl -o daemon.zip https://s3.dualstack.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-linux-3.x.zip
RUN unzip daemon.zip && cp xray /usr/bin/xray
ENTRYPOINT ["/usr/bin/xray", "-t", "0.0.0.0:2000", "-b", "0.0.0.0:2000"]
EXPOSE 2000/udp
EXPOSE 2000/tcp
```

**Note**  
Flags `-t` and `-b` are required to specify a binding address to listen to the loopback of a multi\-container environment\.

**Example Dockerfile – Ubuntu**  
For Debian derivatives, you also need to install certificate authority \(CA\) certificates to avoid issues when downloading the installer\.  

```
FROM ubuntu:16.04
RUN apt-get update && apt-get install -y --force-yes --no-install-recommends apt-transport-https curl ca-certificates wget && apt-get clean && apt-get autoremove && rm -rf /var/lib/apt/lists/*
RUN wget https://s3.dualstack.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.deb
RUN dpkg -i aws-xray-daemon-3.x.deb
ENTRYPOINT ["/usr/bin/xray", "--bind=0.0.0.0:2000", "--bind-tcp=0.0.0.0:2000"]
EXPOSE 2000/udp
EXPOSE 2000/tcp
```

In your task definition, the configuration depends on the networking mode that you use\. Bridge networking is the default and can be used in your default VPC\. In a bridge network, publish UDP port 2000, and create a link from your application container to the daemon container\. Use the `AWS_XRAY_DAEMON_ADDRESS` environment variable to tell the X\-Ray SDK where to send traces\.

**Example Task definition**  

```
    {
      "name": "xray-daemon",
      "image": "123456789012.dkr.ecr.us-east-2.amazonaws.com/xray-daemon",
      "cpu": 32,
      "memoryReservation": 256,
      "portMappings" : [
          {
              "hostPort": 0,
              "containerPort": 2000,
              "protocol": "udp"
          }
       ],
    },
    {
      "name": "scorekeep-api",
      "image": "123456789012.dkr.ecr.us-east-2.amazonaws.com/scorekeep-api",
      "cpu": 192,
      "memoryReservation": 512,
      "environment": [
          { "name" : "AWS_REGION", "value" : "us-east-2" },
          { "name" : "NOTIFICATION_TOPIC", "value" : "arn:aws:sns:us-east-2:123456789012:scorekeep-notifications" },
          { "name" : "AWS_XRAY_DAEMON_ADDRESS", "value" : "xray-daemon:2000" }
      ],
      "portMappings" : [
          {
              "hostPort": 5000,
              "containerPort": 5000
          }
      ],
      "links": [
        "xray-daemon"
      ]
    }
```

If you run your cluster in the private subnet of a VPC, you can use the [`awsvpc` network mode](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html) to attach an elastic network interface \(ENI\) to your containers\. This enables you to avoid using links\. Omit the host port in the port mappings, the link, and the `AWS_XRAY_DAEMON_ADDRESS` environment variable\.

**Example VPC task definition**  

```
{
    "family": "scorekeep",
    "networkMode":"awsvpc",
    "containerDefinitions": [
        {
          "name": "xray-daemon",
          "image": "123456789012.dkr.ecr.us-east-2.amazonaws.com/xray-daemon",
          "cpu": 32,
          "memoryReservation": 256,
          "portMappings" : [
              {
                  "containerPort": 2000,
                  "protocol": "udp"
              }
          ]
        },
        {
            "name": "scorekeep-api",
            "image": "123456789012.dkr.ecr.us-east-2.amazonaws.com/scorekeep-api",
            "cpu": 192,
            "memoryReservation": 512,
            "environment": [
                { "name" : "AWS_REGION", "value" : "us-east-2" },
                { "name" : "NOTIFICATION_TOPIC", "value" : "arn:aws:sns:us-east-2:123456789012:scorekeep-notifications" }
            ],
            "portMappings" : [
                {
                    "containerPort": 5000
                }
            ]
        }
    ]
}
```

## Configure Command Line Options in the Amazon ECS Console<a name="xray-daemon-ecs-cmdline"></a>

In order to set command line options for the container through the Amazon ECS console, there are fields in the task definition that must be\. Command line arguments, in this case, are typically used for local testing, but can also be used for convenience while setting environment variables or to control the startup process\. To learn more about the available command line arguments in X\-Ray, see [Configuring the AWS X\-Ray Daemon](xray-daemon-configuration.md)\.

To set a command line option, you will access your container settings file\. By default this file should be located at `/etc/amazon/xray/cfg.yaml`\. Within the `environemnt` object array, add a comma\-separated list of command line arguments within a string array named `command`\.

The following steps shows a sample where a user wants to set a different account than the one originally instrumented to receive traces\. This sample applies for all implementations of X\-Ray except AWS integrations, such as Lambda, API Gateway, etc\. 

**Requirements**
+ An X\-Ray daemon Docker image\.
+ An Amazon ECS container\.
+ A task definition setup with the X\-Ray daemon\.

**To set X\-Ray to send traces to another account**

1. Ensure that the new resource you want to send traces to has the permissions required to send traces\. For information on permissions, see [Identity and Access Management in AWS X\-Ray](xray-permissions.md)\. 

1. Update the daemon configuration file `/etc/amazon/xray/cfg.yaml` with the ARN of your new resource\.
**Note**  
If you are using the official X\-Ray Docker image, you will need to create a new image with the configuration changes, and have that consume the official base image\. When working with Amazon ECS, you will rerun the configured image as a task\.

1. Restart the daemon, if it is a typical application\. In the case of an Amazon ECS container, you will need to rerun the image with a new task definition\. To learn more, see [Running a Task Using the Fargate Launch Type](         https://docs.aws.amazon.com/AmazonECS/latest/userguide/ecs_run_task_fargate.html)\. 

1. \[How to verify results?\]