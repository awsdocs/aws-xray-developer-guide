# Using AWS X\-Ray with VPC endpoints<a name="xray-security-vpc-endpoint"></a>

If you use Amazon Virtual Private Cloud \(Amazon VPC\) to host your AWS resources, you can establish a private connection between your VPC and X\-Ray\. This enables resources in your Amazon VPC to communicate with the X\-Ray service without going through the public internet\.

Amazon VPC is an AWS service that you can use to launch AWS resources in a virtual network that you define\. With a VPC, you have control over your network settings, such as the IP address range, subnets, route tables, and network gateways\. To connect your VPC to X\-Ray, you define an [interface VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html)\. The endpoint provides reliable, scalable connectivity to X\-Ray without requiring an internet gateway, network address translation \(NAT\) instance, or VPN connection\. For more information, see [What Is Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/) in the *Amazon VPC User Guide*\.

Interface VPC endpoints are powered by AWS PrivateLink, an AWS technology that enables private communication between AWS services by using an elastic network interface with private IP addresses\. For more information, see the [New – AWS PrivateLink for AWS Services](https://aws.amazon.com/blogs/aws/new-aws-privatelink-endpoints-kinesis-ec2-systems-manager-and-elb-apis-in-your-vpc/) blog post and [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/GetStarted.html) in the *Amazon VPC User Guide*\.

To ensure you can create a VPC endpoint for X\-Ray in your chosen AWS Region, see [Supported Regions](#xray-vpc-availability)\. 

## Creating a VPC endpoint for X\-Ray<a name="create-VPC-endpoint-for-xray"></a>

To start using X\-Ray with your VPC, create an interface VPC endpoint for X\-Ray\.

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. Navigate to **Endpoints** within the navigation pane and choose **Create Endpoint**\.

1. Search for and select the name of the AWS X\-Ray service: `com.amazonaws.region.xray`\.  
![\[Select service.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/xray-vpc-select-service.png)

1. Select the VPC you want and then select a subnet in your VPC to use the interface endpoint\. An endpoint network interface is created in the selected subnet\. You can specify more than one subnet in different Availability Zones \(as supported by the service\) to help ensure that your interface endpoint is resilient to Availability Zone failures\. If you do so, an interface network interface is created in each subnet that you specify\.  
![\[Select VPC and subnet.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/xray-vpc-select-vpc.png)

1. \(Optional\) Private DNS is enabled by default for the endpoint, so that you can make requests to X\-Ray using its default DNS hostname\. You can choose to disable it\. 

1. Specify the security groups to associate with the endpoint network interface\.  
![\[Select security groups.\]](http://docs.aws.amazon.com/xray/latest/devguide/images/xray-vpc-select-secgroup.png)

1. \(Optional\) Specify custom policy to control permissions to access the X\-Ray service\. By default, full access is allowed\.

## Controlling access to your X\-Ray VPC endpoint<a name="xray-vpc-endpoint-policy"></a>

A VPC endpoint policy is an IAM resource policy that you attach to an endpoint when you create or modify the endpoint\. If you don't attach a policy when you create an endpoint, Amazon VPC attaches a default policy for you that allows full access to the service\. An endpoint policy doesn't override or replace IAM user policies or service\-specific policies\. It's a separate policy for controlling access from the endpoint to the specified service\. Endpoint policies must be written in JSON format\. For more information, see [Controlling Access to Services with VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-access.html) in the *Amazon VPC User Guide*\.

VPC endpoint policy enables you to control permissions to various X\-Ray actions\. For example, you can create a policy to allow only PutTraceSegment and deny all other actions\. This restricts workloads and services in the VPC to send only trace data to X\-Ray and deny any other action such as retrieve data, change encryption config, or create/update groups\.

The following is an example of an endpoint policy for X\-Ray\. This policy allows users connecting to X\-Ray through the VPC to send segment data to X\-Ray, and also prevents them from performing other X\-Ray actions\.

```
 {"Statement": [
     {"Sid": "Allow PutTraceSegments",
       "Principal": "*",
       "Action": [
         "xray:PutTraceSegments"
       ],
       "Effect": "Allow",
       "Resource": "*"
     }
   ]
 }
```

**To edit the VPC endpoint policy for X\-Ray**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Endpoints**\.

1. If you haven't already created the endpoint for X\-Ray, follow the steps in [Creating a VPC endpoint for X\-Ray](#create-VPC-endpoint-for-xray)\. 

1. Select the **com\.amazonaws\.*region*\.xray** endpoint, and then choose the **Policy** tab\.

1. Choose **Edit Policy**, and then make your changes\.

## Supported Regions<a name="xray-vpc-availability"></a>

X\-Ray currently supports VPC endpoints in the following AWS Regions:
+ US East \(Ohio\)
+ US East \(N\. Virginia\)
+ US West \(N\. California\)
+ US West \(Oregon\)
+ Africa \(Cape Town\)
+ Asia Pacific \(Hong Kong\)
+ Asia Pacific \(Mumbai\)
+ Asia Pacific \(Osaka\) 
+ Asia Pacific \(Seoul\) 
+ Asia Pacific \(Singapore\) 
+ Asia Pacific \(Sydney\)
+ Asia Pacific \(Tokyo\)
+ Canada \(Central\)
+ Europe \(Frankfurt\)
+ Europe \(Ireland\)
+ Europe \(London\)
+ Europe \(Milan\)
+ Europe \(Paris\)
+ Europe \(Stockholm\)
+ Middle East \(Bahrain\)
+ South America \(São Paulo\)
+ AWS GovCloud \(US\-East\)
+ AWS GovCloud \(US\-West\)