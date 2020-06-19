# Running the X\-Ray daemon locally<a name="xray-daemon-local"></a>

You can run the AWS X\-Ray daemon locally on Linux, MacOS, Windows, or in a Docker container\. Run the daemon to relay trace data to X\-Ray when you are developing and testing your instrumented application\. Download and extract the daemon by using the instructions [here](xray-daemon.md#xray-daemon-downloading)\.

When running locally, the daemon can read credentials from an AWS SDK credentials file \(`.aws/credentials` in your user directory\) or from environment variables\. For more information, see [Giving the daemon permission to send data to X\-Ray](xray-daemon.md#xray-daemon-permissions)\.

The daemon listens for UDP data on port 2000\. You can change the port and other options by using a configuration file and command line options\. For more information, see [Configuring the AWS X\-Ray daemon](xray-daemon-configuration.md)\.

## Running the X\-Ray daemon on Linux<a name="xray-daemon-local-linux"></a>

You can run the daemon executable from the command line\. Use the `-o` option to run in local mode, and `-n` to set the region\.

```
~/xray-daemon$ ./xray -o -n us-east-2
```

To run the daemon in the background, use `&`\.

```
~/xray-daemon$ ./xray -o -n us-east-2 &
```

Terminate a daemon process running in the background with `pkill`\.

```
~$ pkill xray
```

## Running the X\-Ray daemon in a Docker container<a name="xray-daemon-local-docker"></a>

To run the daemon locally in a Docker container, save the following text to a file named `Dockerfile`\. Download the complete [example image](https://hub.docker.com/r/amazon/aws-xray-daemon/) on Docker Hub\.

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

Build the container image with `docker build`\.

```
~/xray-daemon$ docker build -t xray-daemon .
```

Run the image in a container with `docker run`\.

```
~/xray-daemon$ docker run \
      --attach STDOUT \
      -v ~/.aws/:/root/.aws/:ro \
      --net=host \
      -e AWS_REGION=us-east-2 \
      --name xray-daemon \
      -p 2000:2000/udp \
      xray-daemon -o
```

This command uses the following options:
+ `--attach STDOUT` – View output from the daemon in the terminal\.
+ `-v ~/.aws/:/root/.aws/:ro` – Give the container read\-only access to the `.aws` directory to let it read your AWS SDK credentials\.
+ `AWS_REGION=us-east-2` – Set the `AWS_REGION` environment variable to tell the daemon which region to use\.
+ `--net=host` – Attach the container to the `host` network\. Containers on the host network can communicate with each other without publishing ports\.
+ `-p 2000:2000/udp` – Map UDP port 2000 on your machine to the same port on the container\. This is not required for containers on the same network to communicate, but it does let you send segments to the daemon [from the command line](xray-api-sendingdata.md#xray-api-daemon) or from an application not running in Docker\.
+ `--name xray-daemon` – Name the container `xray-daemon` instead of generating a random name\.
+ `-o` \(after the image name\) – Append the `-o` option to the entry point that runs the daemon within the container\. This option tells the daemon to run in local mode to prevent it from trying to read Amazon EC2 instance metadata\.

To stop the daemon, use `docker stop`\. If you make changes to the `Dockerfile` and build a new image, you need to delete the existing container before you can create another one with the same name\. Use `docker rm` to delete the container\.

```
$ docker stop xray-daemon
$ docker rm xray-daemon
```

The Scorekeep sample application shows how to use the X\-Ray daemon in a local Docker container\. See [Instrumenting Amazon ECS applications](scorekeep-ecs.md) for details\.

## Running the X\-Ray daemon on Windows<a name="xray-daemon-local-windows"></a>

You can run the daemon executable from the command line\. Use the `-o` option to run in local mode, and `-n` to set the region\.

```
> .\xray_windows.exe -o -n us-east-2
```

Use a PowerShell script to create and run a service for the daemon\.

**Example PowerShell script \- Windows**  

```
if ( Get-Service "AWSXRayDaemon" -ErrorAction SilentlyContinue ){
    sc.exe stop AWSXRayDaemon
    sc.exe delete AWSXRayDaemon
}
if ( Get-Item -path aws-xray-daemon -ErrorAction SilentlyContinue ) {
    Remove-Item -Recurse -Force aws-xray-daemon
}

$currentLocation = Get-Location
$zipFileName = "aws-xray-daemon-windows-service-3.x.zip"
$zipPath = "$currentLocation\$zipFileName"
$destPath = "$currentLocation\aws-xray-daemon"
$daemonPath = "$destPath\xray.exe"
$daemonLogPath = "C:\inetpub\wwwroot\xray-daemon.log"
$url = "https://s3.dualstack.us-west-2.amazonaws.com/aws-xray-assets.us-west-2/xray-daemon/aws-xray-daemon-windows-service-3.x.zip"

Invoke-WebRequest -Uri $url -OutFile $zipPath
Add-Type -Assembly "System.IO.Compression.Filesystem"
[io.compression.zipfile]::ExtractToDirectory($zipPath, $destPath)

sc.exe create AWSXRayDaemon binPath= "$daemonPath -f $daemonLogPath"
sc.exe start AWSXRayDaemon
```

## Running the X\-Ray daemon on OS X<a name="xray-daemon-local-osx"></a>

You can run the daemon executable from the command line\. Use the `-o` option to run in local mode, and `-n` to set the region\.

```
~/xray-daemon$ ./xray_mac -o -n us-east-2
```

To run the daemon in the background, use `&`\.

```
~/xray-daemon$ ./xray_mac -o -n us-east-2 &
```

Use `nohup` to prevent the daemon from terminating when the terminal is closed\.

```
~/xray-daemon$ nohup ./xray_mac &
```