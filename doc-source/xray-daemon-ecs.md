# Running the X\-Ray Daemon on Amazon ECS<a name="xray-daemon-ecs"></a>

On Amazon ECS, create a Docker image that runs the daemon, upload it to a Docker image repository, and then deploy it to your Amazon ECS cluster\.

Use an instance profile to grant the daemon permission to upload trace data to X\-Ray\. For more information, see \.

**Example Dockerfile**  
Add the following lines to your Dockerfile file to copy and run the daemon\.  

```
FROM amazonlinux
COPY xray /usr/bin/xray
ENTRYPOINT ["/usr/bin/xray", "log-file /var/log/xray-daemon.log"]
```

Download the X\-Ray daemon Linux executable into the same folder as your Dockerfile and build it to create an image\.

```
$ wget https://s3.dualstack.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-linux-2.x.zip
$ unzip aws-xray-daemon-linux-2.x.zip
$ docker build
```

To run the daemon in its own container, you need to change the address on which the daemon runs to make it routable from other containers\. Use the `--bind` option to run the daemon on `0.0.0.0` instead of `localhost`\.

**Example Dockerfile \- Ubuntu**  
This Dockerfile file runs the daemon in its own container\. It downloads the Debian installer from Amazon S3 and installs it with `dpkg`\. For Debian derivatives, you also need to install certificate authority \(CA\) certificates to avoid issues when downloading the installer\.  

```
FROM ubuntu:16.04

# Install CA certificates
RUN apt-get update && apt-get install -y --force-yes --no-install-recommends apt-transport-https curl ca-certificates wget && apt-get clean && apt-get autoremove && rm -rf /var/lib/apt/lists/*

RUN wget https://s3.dualstack.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-2.x.deb

RUN dpkg -i aws-xray-daemon-2.x.deb

# Run the X-Ray daemon
CMD ["/usr/bin/xray", "--bind=0.0.0.0:2000"]
```