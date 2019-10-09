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