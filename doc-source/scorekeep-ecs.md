# Instrumenting Amazon ECS applications<a name="scorekeep-ecs"></a>

In the [https://github.com/awslabs/eb-java-scorekeep/tree/xray-ecs](https://github.com/awslabs/eb-java-scorekeep/tree/xray-ecs) branch, the Scorekeep sample application shows how to instrument an application running in Amazon Elastic Container Service \(Amazon ECS\)\. The branch provides scripts and configuration files for creating, uploading, and running Docker images in a Multicontainer Docker environment in AWS Elastic Beanstalk\.

The project includes three Dockerfiles that define container images for the API, front end, and X\-Ray daemon components\.
+ `/Dockerfile` – The Scorekeep API\.
+ `/scorekeep-frontend/Dockerfile` – The Angular web app client, and the nginx proxy that routes incoming traffic\.
+ `/xray-daemon/Dockerfile` – The X\-Ray daemon\.

The X\-Ray daemon Dockerfile creates an image based on Amazon Linux that runs the X\-Ray daemon\. Download the complete [example image](https://hub.docker.com/r/amazon/aws-xray-daemon/) on Docker Hub\.

**Example Dockerfile – Amazon Linux**  

```
FROM amazonlinux
RUN yum install -y unzip
RUN curl -o daemon.zip https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-linux-3.x.zip
RUN unzip daemon.zip && cp xray /usr/bin/xray
ENTRYPOINT ["/usr/bin/xray", "-t", "0.0.0.0:2000", "-b", "0.0.0.0:2000"]
EXPOSE 2000/udp
EXPOSE 2000/tcp
```

The makefile in the same directory defines commands for building the image, uploading it to Amazon ECR, and [running it locally](xray-daemon-local.md#xray-daemon-local-docker)\.

To run the containers on Amazon ECS, the branch includes a script to generate a `Dockerrun.aws.json` file, which you can deploy to a multicontainer Docker environment in Elastic Beanstalk\. The [template](https://github.com/awslabs/eb-java-scorekeep/tree/xray-ecs/task-definition/template/scorekeep-dockerrun.template) that the script uses shows how to write a task definition that configures [networking between the containers in Amazon ECS](xray-daemon-ecs.md)\. 

**Note**  
`Dockerrun.aws.json` is the Elastic Beanstalk version of an Amazon ECS task definition file\. If you don't want to use Elastic Beanstalk to create your Amazon ECS cluster, you can modify the `Dockerrun.aws.json` file to run on Amazon ECS directly by removing the `AWSEBDockerrunVersion` key from the file\.

See the [branch README](https://github.com/awslabs/eb-java-scorekeep/tree/xray-ecs) for instructions on how to deploy Scorekeep to Amazon ECS\.