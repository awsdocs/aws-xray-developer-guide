# Running the X\-Ray daemon on AWS Elastic Beanstalk<a name="xray-daemon-beanstalk"></a>

To relay trace data from your application to AWS X\-Ray, you can run the X\-Ray daemon on your Elastic Beanstalk environment's Amazon EC2 instances\. For a list of supported platforms, see [Configuring AWS X\-Ray Debugging](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-configuration-debugging.html) in the *AWS Elastic Beanstalk Developer Guide*\.

**Note**  
The daemon uses your environment's instance profile for permissions\. For instructions about adding permissions to the Elastic Beanstalk instance profile, see [Giving the daemon permission to send data to X\-Ray](xray-daemon.md#xray-daemon-permissions)\.

Elastic Beanstalk platforms provide a configuration option that you can set to run the daemon automatically\. You can enable the daemon in a configuration file in your source code or by choosing an option in the Elastic Beanstalk console\. When you enable the configuration option, the daemon is installed on the instance and runs as a service\.

The version included on Elastic Beanstalk platforms might not be the latest version\. See the [Supported Platforms topic](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts.platforms.html) to find out the version of the daemon that is available for your platform configuration\.

Elastic Beanstalk does not provide the X\-Ray daemon on the Multicontainer Docker \(Amazon ECS\) platform\. The Scorekeep sample application shows how to use the X\-Ray daemon on Amazon ECS with Elastic Beanstalk\. See [Instrumenting Amazon ECS applications](scorekeep-ecs.md) for details\.

## Using the Elastic Beanstalk X\-Ray integration to run the X\-Ray daemon<a name="xray-daemon-beanstalk-option"></a>

Use the console to turn on X\-Ray integration, or configure it in your application source code with a configuration file\.

**To enable the X\-Ray daemon in the Elastic Beanstalk console**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-console.html) for your environment\.

1. Choose **Configuration**\.

1. Choose **Software Settings**\.

1. For **X\-Ray daemon**, choose **Enabled**\.

1. Choose **Apply**\.

You can include a configuration file in your source code to make your configuration portable between environments\.

**Example \.ebextensions/xray\-daemon\.config**  

```
option_settings:
  aws:elasticbeanstalk:xray:
    XRayEnabled: true
```

Elastic Beanstalk passes a configuration file to the daemon and outputs logs to a standard location\.

**On Windows Server Platforms**
+ **Configuration file** – `C:\Progam Files\Amazon\XRay\cfg.yaml`
+ **Logs** – `c:\Program Files\Amazon\XRay\logs\xray-service.log`

**On Linux Platforms**
+ **Configuration file** – `/etc/amazon/xray/cfg.yaml`
+ **Logs** – `/var/log/xray/xray.log`

Elastic Beanstalk provides tools for pulling instance logs from the AWS Management Console or command line\. You can tell Elastic Beanstalk to include the X\-Ray daemon logs by adding a task with a configuration file\.

**Example \.ebextensions/xray\-logs\.config \- Linux**  

```
files:
  "/opt/elasticbeanstalk/tasks/taillogs.d/xray-daemon.conf" :
    mode: "000644"
    owner: root
    group: root
    content: |
      /var/log/xray/xray.log
```

**Example \.ebextensions/xray\-logs\.config \- Windows server**  

```
files:
  "c:/Program Files/Amazon/ElasticBeanstalk/config/taillogs.d/xray-daemon.conf" :
    mode: "000644"
    owner: root
    group: root
    content: |
      c:\Progam Files\Amazon\XRay\logs\xray-service.log
```

See [Viewing Logs from Your Elastic Beanstalk Environment's Amazon EC2 Instances](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.logging.html) in the *AWS Elastic Beanstalk Developer Guide* for more information\.

## Downloading and running the X\-Ray daemon manually \(advanced\)<a name="xray-daemon-beanstalk-manual"></a>

If the X\-Ray daemon isn't available for your platform configuration, you can download it from Amazon S3 and run it with a configuration file\.

Use an Elastic Beanstalk configuration file to download and run the daemon\.

**Example \.ebextensions/xray\.config \- Linux**  

```
commands:
  01-stop-tracing:
    command: yum remove -y xray
    ignoreErrors: true
  02-copy-tracing:
    command: curl https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.rpm -o /home/ec2-user/xray.rpm
  03-start-tracing:
    command: yum install -y /home/ec2-user/xray.rpm

files:
  "/opt/elasticbeanstalk/tasks/taillogs.d/xray-daemon.conf" :
    mode: "000644"
    owner: root
    group: root
    content: |
      /var/log/xray/xray.log
  "/etc/amazon/xray/cfg.yaml" :
    mode: "000644"
    owner: root
    group: root
    content: |
      Logging:
        LogLevel: "debug"
      Version: 2
```

**Example \.ebextensions/xray\.config \- Windows server**  

```
container_commands:
  01-execute-config-script:
    command: Powershell.exe -ExecutionPolicy Bypass -File c:\\temp\\installDaemon.ps1
    waitAfterCompletion: 0
 
files:
  "c:/temp/installDaemon.ps1":
    content: |
      if ( Get-Service "AWSXRayDaemon" -ErrorAction SilentlyContinue ) {
          sc.exe stop AWSXRayDaemon
          sc.exe delete AWSXRayDaemon
      }

      $targetLocation = "C:\Program Files\Amazon\XRay"
      if ((Test-Path $targetLocation) -eq 0) {
          mkdir $targetLocation
      }

      $zipFileName = "aws-xray-daemon-windows-service-3.x.zip"
      $zipPath = "$targetLocation\$zipFileName"
      $destPath = "$targetLocation\aws-xray-daemon"
      if ((Test-Path $destPath) -eq 1) {
          Remove-Item -Recurse -Force $destPath
      }

      $daemonPath = "$destPath\xray.exe"
      $daemonLogPath = "$targetLocation\xray-daemon.log"
      $url = "https://s3.dualstack.us-west-2.amazonaws.com/aws-xray-assets.us-west-2/xray-daemon/aws-xray-daemon-windows-service-3.x.zip"

      Invoke-WebRequest -Uri $url -OutFile $zipPath
      Add-Type -Assembly "System.IO.Compression.Filesystem"
      [io.compression.zipfile]::ExtractToDirectory($zipPath, $destPath)

      New-Service -Name "AWSXRayDaemon" -StartupType Automatic -BinaryPathName "`"$daemonPath`" -f `"$daemonLogPath`""
      sc.exe start AWSXRayDaemon
    encoding: plain
  "c:/Program Files/Amazon/ElasticBeanstalk/config/taillogs.d/xray-daemon.conf" :
    mode: "000644"
    owner: root
    group: root
    content: |
      C:\Program Files\Amazon\XRay\xray-daemon.log
```

These examples also add the daemon's log file to the Elastic Beanstalk tail logs task, so that it's included when you request logs with the console or Elastic Beanstalk Command Line Interface \(EB CLI\)\.