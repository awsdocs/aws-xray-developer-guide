# Configuring the AWS X\-Ray daemon<a name="xray-daemon-configuration"></a>

You can use command line options or a configuration file to customize the X\-Ray daemon's behavior\. Most options are available using both methods, but some are only available in configuration files and some only at the command line\.

To get started, the only option that you need to know is `-n` or `--region`, which you use to set the region that the daemon uses to send trace data to X\-Ray\.

```
~/xray-daemon$ ./xray -n us-east-2
```

If you are running the daemon locally, that is, not on Amazon EC2, you can add the `-o` option to skip checking for instance profile credentials so the daemon will become ready more quickly\.

```
~/xray-daemon$ ./xray -o -n us-east-2
```

The rest of the command line options let you configure logging, listen on a different port, limit the amount of memory that the daemon can use, or assume a role to send trace data to a different account\.

You can pass a configuration file to the daemon to access advanced configuration options and do things like limit the number of concurrent calls to X\-Ray, disable log rotation, and send traffic to a proxy\.

**Topics**
+ [Supported environment variables](#xray-daemon-configuration-variables)
+ [Using command line options](#xray-daemon-configuration-commandline)
+ [Using a configuration file](#xray-daemon-configuration-configfile)

## Supported environment variables<a name="xray-daemon-configuration-variables"></a>

The X\-Ray daemon supports the following environment variables:
+ `AWS_REGION` – Specifies the [AWS Region](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-quick-configuration-region) of the X\-Ray service endpoint\. 
+ `HTTPS_PROXY` – Specifies a proxy address for the daemon to upload segments through\. This can be either the DNS domain names or IP addresses and port numbers used by your proxy servers\.

## Using command line options<a name="xray-daemon-configuration-commandline"></a>

Pass these options to the daemon when you run it locally or with a user data script\.

**Command Line Options**
+ `-b`, `--bind` – Listen for segment documents on a different UDP port\.

  ```
  --bind "127.0.0.1:3000"
  ```

  Default – `2000`\.
+ `-t`, `--bind-tcp` – Listen for calls to the X\-Ray service on a different TCP port\.

  ```
  -bind-tcp "127.0.0.1:3000"
  ```

  Default – `2000`\.
+ `-c`, `--config` – Load a configuration file from the specified path\.

  ```
  --config "/home/ec2-user/xray-daemon.yaml"
  ```
+ `-f`, `--log-file` – Output logs to the specified file path\.

  ```
  --log-file "/var/log/xray-daemon.log"
  ```
+ `-l`, `--log-level` – Log level, from most verbose to least: dev, debug, info, warn, error, prod\.

  ```
  --log-level warn
  ```

  Default – `prod`
+ `-m`, `--buffer-memory` – Change the amount of memory in MB that buffers can use \(minimum 3\)\.

  ```
  --buffer-memory 50
  ```

  Default – 1% of available memory\.
+ `-o`, `--local-mode` – Don't check for EC2 instance metadata\.
+ `-r`, `--role-arn` – Assume the specified IAM role to upload segments to a different account\.

  ```
  --role-arn "arn:aws:iam::123456789012:role/xray-cross-account"
  ```
+ `-a`, `--resource-arn` – Amazon Resource Name \(ARN\) of the AWS resource running the daemon\.
+ `-p`, `--proxy-address` – Upload segments to AWS X\-Ray through a proxy\. The proxy server's protocol must be specified\.

  ```
  --proxy-address "http://192.0.2.0:3000"
  ```
+ `-n`, `--region` – Send segments to X\-Ray service in a specific region\.
+ `-v`, `--version` – Show AWS X\-Ray daemon version\.
+ `-h`, `--help` – Show the help screen\.

## Using a configuration file<a name="xray-daemon-configuration-configfile"></a>

You can also use a YAML format file to configure the daemon\. Pass the configuration file to the daemon by using the `-c` option\.

```
~$ ./xray -c ~/xray-daemon.yaml
```

**Configuration file options**
+ `TotalBufferSizeMB` – Maximum buffer size in MB \(minimum 3\)\. Choose 0 to use 1% of host memory\.
+ `Concurrency` – Maximum number of concurrent calls to AWS X\-Ray to upload segment documents\.
+ `Region` – Send segments to AWS X\-Ray service in a specific region\.
+ `Socket` – Configure the daemon's binding\.
  + `UDPAddress` – Change the port on which the daemon listens\.
  + `TCPAddress` – Listen for [calls to the X\-Ray service](xray-api-sampling.md) on a different TCP port\.
+ `Logging` – Configure logging behavior\.
  + `LogRotation` – Set to `false` to disable log rotation\.
  + `LogLevel` – Change the log level, from most verbose to least: `dev`, `debug`, `info`, `warn`, `error`, `prod` \(default\)\.
  + `LogPath` – Output logs to the specified file path\.
+ `LocalMode` – Set to `true` to skip checking for EC2 instance metadata\.
+ `ResourceARN` – Amazon Resource Name \(ARN\) of the AWS resource running the daemon\.
+ `RoleARN` – Assume the specified IAM role to upload segments to a different account\.
+ `ProxyAddress` – Upload segments to AWS X\-Ray through a proxy\.
+ `Endpoint` – Change the X\-Ray service endpoint to which the daemon sends segment documents\.
+ `NoVerifySSL` – Disable TLS certificate verification\.
+ `Version` – Daemon configuration file format version\.

**Example Xray\-daemon\.yaml**  
This configuration file changes the daemon's listening port to 3000, turns off checks for instance metadata, sets a role to use for uploading segments, and changes region and logging options\.  

```
Socket:
  UDPAddress: "127.0.0.1:3000"
  TCPAddress: "127.0.0.1:3000"
Region: "us-west-2"
Logging:
  LogLevel: "warn"
  LogPath: "/var/log/xray-daemon.log"
LocalMode: true
RoleARN: "arn:aws:iam::123456789012:role/xray-cross-account"
Version: 2
```