# Using Amazon Kinesis Data Streams with Interface VPC Endpoints<a name="vpc"></a>

You can use an interface VPC endpoint to keep traffic between your Amazon VPC and Kinesis Data Streams from leaving the Amazon network\. Interface VPC endpoints don't require an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection\. Interface VPC endpoints are powered by AWS PrivateLink, an AWS technology that enables private communication between AWS services using an elastic network interface with private IPs in your Amazon VPC\. For more information, see [Amazon Virtual Private Cloud](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html) and [Interface VPC Endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint)\. 

**Topics**
+ [Using Interface VPC Endpoints for Kinesis Data Streams](#using-interface-vpc-endpoints)
+ [Controlling Access to VPCE Endpoints for Kinesis Data Streams](#interface-vpc-endpoints-policies)
+ [Availability of VPC Endpoint Policies for Kinesis Data Streams](#availability)

## Using Interface VPC Endpoints for Kinesis Data Streams<a name="using-interface-vpc-endpoints"></a>

To get started you do not need to change the settings for your streams, producers, or consumers\. Simply create an interface VPC endpoint in order for your Kinesis Data Streams traffic from and to your Amazon VPC resources to start flowing through the interface VPC endpoint\. For more information, see [Creating an Interface Endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint)\.

The Kinesis Producer Library \(KPL\) and Kinesis Consumer Library \(KCL\) call AWS services like Amazon CloudWatch and Amazon DynamoDB using either public endpoints or private interface VPC endpoints, whichever are in use\. For example, if your KCL application is running in a VPC with DynamoDB interface with VPC endpoints enabled, calls between DynamoDB and your KCL application are flowing through the interface VPC endpoint\.

## Controlling Access to VPCE Endpoints for Kinesis Data Streams<a name="interface-vpc-endpoints-policies"></a>

VPC endpoint policies enable you to control access by either attaching a policy to a VPC endpoint or by using additional fields in a policy that is attached to an IAM user, group, or role to restrict access to only occur via the specified VPC endpoint\. These policies can be used to restrict access to specific streams to a specified VPC endpoint when used in conjunction with the IAM policies to only grant access to Kinesis data stream actions via the specified VPC endpoint\.

The following are example endpoint policies for accessing Kinesis data streams\.
+ **VPC policy example: read\-only access** \- this sample policy can be attached to a VPC endpoint\. \(For more information, see [Controlling Access to Amazon VPC Resources](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_IAM.html)\)\. It restricts actions to only listing and describing a Kinesis data stream through the VPC endpoint to which it is attached\.

  ```
  {
    "Statement": [
      {
        "Sid": "ReadOnly",
        "Principal": "*",
        "Action": [
          "kinesis:List*",
          "kinesis:Describe*"
        ],
        "Effect": "Allow",
        "Resource": "*"
      }
    ]
  }
  ```
+ **VPC policy example: restrict access to a specific Kinesis data stream** \- this sample policy can be attached to a VPC endpoint\. It restricts access to a specific data stream through the VPC endpoint to which it is attached\.

  ```
  {
    "Statement": [
      {
        "Sid": "AccessToSpecificDataStream",
        "Principal": "*",
        "Action": "kinesis:*",
        "Effect": "Allow",
        "Resource": "arn:aws:kinesis:us-east-1:123456789012:stream/MyStream"
      }
    ]
  }
  ```
+ **IAM policy example: restrict access to a specific Stream from a specific VPC endpoint only** \- this sample policy can be attached to an IAM user, role, or group\. It restricts access to a specified Kinesis data stream to occur only from a specified VPC endpoint\.

  ```
  {
     "Version": "2012-10-17",
     "Statement": [
        {
           "Sid": "AccessFromSpecificEndpoint",
           "Action": "kinesis:*",
           "Effect": "Deny",
           "Resource": "arn:aws:kinesis:us-east-1:123456789012:stream/MyStream",
           "Condition": { "StringNotEquals" : { "aws:sourceVpce": "vpce-11aa22bb" } }
        }
     ]
  }
  ```

## Availability of VPC Endpoint Policies for Kinesis Data Streams<a name="availability"></a>

Kinesis Data Streams interface VPC endpoints with policies are supported in the following regions: 
+ US West \(Oregon\)
+ Europe \(Paris\)
+ Europe \(Ireland\)
+ US East \(N\. Virginia\)
+ Europe \(Stockholm\)
+ Asia Pacific \(Mumbai\)
+ US East \(Ohio\)
+ Europe \(Frankfurt\)
+ South America \(São Paulo\)
+ Asia Pacific \(Seoul\)
+ Europe \(London\)
+ Asia Pacific \(Tokyo\)
+ US West \(N\. California\)
+ Asia Pacific \(Singapore\)
+ Asia Pacific \(Sydney\)
+ Canada \(Central\)
+ China \(Beijing\)
+ China \(Ningxia\)
+ Asia Pacific \(Hong Kong\)
+ Middle East \(Bahrain\)
+ Europe \(Milan\)
+ Africa \(Cape Town\)