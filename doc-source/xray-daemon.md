# AWS X\-Ray daemon<a name="xray-daemon"></a>

The AWS X\-Ray daemon is a software application that listens for traffic on UDP port 2000, gathers raw segment data, and relays it to the AWS X\-Ray API\. The daemon works in conjunction with the AWS X\-Ray SDKs and must be running so that data sent by the SDKs can reach the X\-Ray service\.

**Note**  
The X\-Ray daemon is an open source project\. You can follow the project and submit issues and pull requests on GitHub: [github\.com/aws/aws\-xray\-daemon](https://github.com/aws/aws-xray-daemon)

On AWS Lambda and AWS Elastic Beanstalk, use those services' integration with X\-Ray to run the daemon\. Lambda runs the daemon automatically any time a function is invoked for a sampled request\. On Elastic Beanstalk, [use the `XRayEnabled` configuration option](xray-daemon-beanstalk.md) to run the daemon on the instances in your environment\.

To run the X\-Ray daemon locally, on\-premises, or on other AWS services, [download it from Amazon S3](#xray-daemon-downloading), [run it](#xray-daemon-running), and then [give it permission](#xray-daemon-permissions) to upload segment documents to X\-Ray\.

## Downloading the daemon<a name="xray-daemon-downloading"></a>

You can download the daemon from Amazon S3 to run it locally, or to install it on an Amazon EC2 instance on launch\.

**X\-Ray daemon installers and executables**
+ **Linux \(executable\)** – [https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-linux-3.x.zip](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-linux-3.x.zip) \([sig](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-linux-3.x.zip.sig)\)
+ **Linux \(RPM installer\)** – [https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.rpm](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.rpm)
+ **Linux \(DEB installer\)** – [https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.deb](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.deb)
+ **OS X \(executable\)** – [https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-macos-3.x.zip](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-macos-3.x.zip) \([sig](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-macos-3.x.zip.sig)\) 
+ **Windows \(executable\)** – [https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-windows-process-3.x.zip](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-windows-process-3.x.zip) \([sig](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-windows-process-3.x.zip.sig)\)
+ **Windows \(service\) ** – [https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-windows-service-3.x.zip](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-windows-service-3.x.zip) \([sig](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-windows-service-3.x.zip.sig)\)

These links always point to the latest release of the daemon\. To download a specific release, replace `3.x` with the version number\. For example, `2.1.0`\.

X\-Ray assets are replicated to buckets in every supported region\. To use the bucket closest to you or your AWS resources, replace the region in the above links with your region\.

## Verifying the daemon archive's signature<a name="xray-daemon-signature"></a>

GPG signature files are included for daemon assets compressed in ZIP archives\. The public key is here: [https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray.gpg](https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray.gpg)\.

You can use the public key to verify that the daemon's ZIP archive is original and unmodified\. First, import the public key with [GnuPG](https://gnupg.org/index.html)\.

**To import the public key**

1. Download the public key\.

1. Import the public key into your keyring\.

   ```
   $ gpg --import aws-xray.gpg
   gpg: /Users/me/.gnupg/trustdb.gpg: trustdb created
   gpg: key 7BFE036BFE6157D3: public key "AWS X-Ray <aws-xray@amazon.com>" imported
   gpg: Total number processed: 1
   gpg:               imported: 1
   ```

Use the imported key to verify the signature of the daemon's ZIP archive\.

**To verify an archive's signature**

1. Download the archive and signature file\.

1. Run `gpg --verify` to verify the signature\.

   ```
   $ gpg --verify aws-xray-daemon-linux-3.0.0.zip.sig aws-xray-daemon-linux-3.0.0.zip
   gpg: Signature made Wed 19 Apr 2017 05:06:31 AM UTC using RSA key ID FE6157D3
   gpg: Good signature from "AWS X-Ray <aws-xray@amazon.com>"
   gpg: WARNING: This key is not certified with a trusted signature!
   gpg:          There is no indication that the signature belongs to the owner.
   Primary key fingerprint: EA6D 9271 FBF3 6990 277F  4B87 7BFE 036B FE61 57D3
   ```

Note the warning about trust\. A key is only trusted if you or someone you trust has signed it\. This does not mean that the signature is invalid, only that you have not verified the public key\.

## Running the daemon<a name="xray-daemon-running"></a>

Run the daemon locally from the command line\. Use the `-o` option to run in local mode, and `-n` to set the region\.

```
~/Downloads$ ./xray -o -n us-east-2
```

For detailed platform\-specific instructions, see the following topics:
+ **Linux \(local\)** – [Running the X\-Ray daemon on Linux](xray-daemon-local.md#xray-daemon-local-linux)
+ **Windows \(local\)** – [Running the X\-Ray daemon on Windows](xray-daemon-local.md#xray-daemon-local-windows)
+ **Elastic Beanstalk** – [Running the X\-Ray daemon on AWS Elastic Beanstalk](xray-daemon-beanstalk.md)
+ **Amazon EC2** – [Running the X\-Ray daemon on Amazon EC2](xray-daemon-ec2.md)
+ **Amazon ECS** – [Running the X\-Ray daemon on Amazon ECS](xray-daemon-ecs.md)

You can customize the daemon's behavior further by using command line options or a configuration file\. See [Configuring the AWS X\-Ray daemon](xray-daemon-configuration.md) for details\.

## Giving the daemon permission to send data to X\-Ray<a name="xray-daemon-permissions"></a>

The X\-Ray daemon uses the AWS SDK to upload trace data to X\-Ray, and it needs AWS credentials with permission to do that\.

On Amazon EC2, the daemon uses the instance's instance profile role automatically\. Locally, save your access keys to a file named `credentials` in your user directory under a folder named `.aws`\.

**Example \~/\.aws/credentials**  

```
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

If you specify credentials in more than one location \(credentials file, instance profile, or environment variables\), the SDK provider chain determines which credentials are used\. For more information about providing credentials to the SDK, see [Specifying Credentials](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html#specifying-credentials) in the *AWS SDK for Go Developer Guide*\.

The IAM role or user that the daemon's credentials belong to must have permission to write data to the service on your behalf\.
+ To use the daemon on Amazon EC2, create a new instance profile role or add the managed policy to an existing one\.
+ To use the daemon on Elastic Beanstalk, add the managed policy to the Elastic Beanstalk default instance profile role\.
+ To run the daemon locally, create an IAM user and save its access keys on your computer\.

For more information, see [Identity and access management for AWS X\-Ray](security-iam.md)\.

## X\-Ray daemon logs<a name="xray-daemon-logging"></a>

The daemon outputs information about its current configuration and segments that it sends to AWS X\-Ray\.

```
2016-11-24T06:07:06Z [Info] Initializing AWS X-Ray daemon 2.1.0
2016-11-24T06:07:06Z [Info] Using memory limit of 49 MB
2016-11-24T06:07:06Z [Info] 313 segment buffers allocated
2016-11-24T06:07:08Z [Info] Successfully sent batch of 1 segments (0.123 seconds)
2016-11-24T06:07:09Z [Info] Successfully sent batch of 1 segments (0.006 seconds)
```

By default, the daemon outputs logs to STDOUT\. If you run the daemon in the background, use the `--log-file` command line option or a configuration file to set the log file path\. You can also set the log level and disable log rotation\. See [Configuring the AWS X\-Ray daemon](xray-daemon-configuration.md) for instructions\.

On Elastic Beanstalk, the platform sets the location of the daemon logs\. See [Running the X\-Ray daemon on AWS Elastic Beanstalk](xray-daemon-beanstalk.md) for details\.