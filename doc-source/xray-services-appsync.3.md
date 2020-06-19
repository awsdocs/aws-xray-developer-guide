# AWS AppSync and AWS X\-Ray<a name="xray-services-appsync"></a>

You can enable and trace requests for AWS AppSync\. For more information, see [Tracing with AWS X\-Ray](https://docs.aws.amazon.com/appsync/latest/devguide/x-ray-tracing.html) for instructions\.

When X\-Ray tracing is enabled for an AWS AppSync API, an AWS Identity and Access Management [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) is automatically created in your account with the appropriate permissions\. This allows AWS AppSync to send traces to X\-Ray in a secure way\.