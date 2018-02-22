# Running the X\-Ray Daemon Locally<a name="xray-daemon-local"></a>

You can run the daemon locally for development and testing\.

When running locally, the daemon can read credentials from an AWS SDK credentials file \(`.aws/credentials` in your user directory\) or from environment variables\. For more information, see [Giving the Daemon Permission to Send Data to X\-Ray](xray-daemon.md#xray-daemon-permissions)\.

The daemon listens for UDP data on port 2000\. You can change the port and other options by using a configuration file and command line options\. For more information, see [Configuring the AWS X\-Ray Daemon](xray-daemon-configuration.md)\.

## Running the X\-Ray Daemon on Linux<a name="xray-daemon-local-linux"></a>

You can run the daemon executable from the command line, as follows\.

```
~/xray-daemon$ ./xray
```

To run the daemon in the background, use `&`\.

```
~/xray-daemon$ ./xray &
```

Terminate a daemon process running in the background with `pkill`\.

```
~$ pkill xray
```

## Running the X\-Ray Daemon on Windows<a name="xray-daemon-local-windows"></a>

You can run the daemon executable from the command line\.

```
> .\xray_windows.exe
```

Use a PowerShell script to create and run a service for the daemon\.

**Example PowerShell Script \- Windows**  

```
if ( Get-Service "AWSXRayDaemon" -ErrorAction SilentlyContinue ){
    sc.exe stop AWSXRayDaemon
    sc.exe delete AWSXRayDaemon
}
if ( Get-Item -path aws-xray-daemon -ErrorAction SilentlyContinue ) {
    Remove-Item -Recurse -Force aws-xray-daemon
}

$currentLocation = Get-Location
$zipFileName = "aws-xray-daemon-windows-service-2.x.zip"
$zipPath = "$currentLocation\$zipFileName"
$destPath = "$currentLocation\aws-xray-daemon"
$daemonPath = "$destPath\xray.exe"
$daemonLogPath = "C:\inetpub\wwwroot\xray-daemon.log"
$url = "https://s3.dualstack.us-west-2.amazonaws.com/aws-xray-assets.us-west-2/xray-daemon/aws-xray-daemon-windows-service-2.x.zip"

Invoke-WebRequest -Uri $url -OutFile $zipPath
Add-Type -Assembly "System.IO.Compression.Filesystem"
[io.compression.zipfile]::ExtractToDirectory($zipPath, $destPath)

sc.exe create AWSXRayDaemon binPath= "$daemonPath -f $daemonLogPath"
sc.exe start AWSXRayDaemon
```

## Running the X\-Ray Daemon on OS X<a name="xray-daemon-local-osx"></a>

You can run the daemon executable from the command line\.

```
~/xray-daemon$ ./xray_mac
```

To run the daemon in the background, use `&`\.

```
~/xray-daemon$ ./xray_mac &
```

Use `nohup` to prevent the daemon from terminating when the terminal is closed\.

```
~/xray-daemon$ nohup ./xray_mac &
```