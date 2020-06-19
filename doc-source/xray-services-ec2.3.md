# Amazon Elastic Compute Cloud and AWS X\-Ray<a name="xray-services-ec2"></a>

You can install and run the X\-Ray daemon on an Amazon EC2 instance with a user data script\. See [Running the X\-Ray daemon on Amazon EC2](xray-daemon-ec2.md) for instructions\.

Use an instance profile to grant the daemon permission to upload trace data to X\-Ray\. For more information, see [Giving the daemon permission to send data to X\-Ray](xray-daemon.md#xray-daemon-permissions)\.