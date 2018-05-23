# Using Amazon Kinesis Data Streams with Interface VPC Endpoints<a name="vpc"></a>

## Interface VPC endpoints for Kinesis Data Streams<a name="interface-vpc-endpoints"></a>

You can use an interface VPC endpoint to keep traffic between your Amazon VPC and Kinesis Data Streams from leaving the Amazon network\. Interface VPC endpoints don't require an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection\. Interface VPC endpoints are powered by AWS PrivateLink, an AWS technology that enables private communication between AWS services using an elastic network interface with private IPs in your Amazon VPC\. For more information, see [Amazon Virtual Private Cloud](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html)\. 

## Using interface VPC endpoints for Kinesis Data Streams<a name="using-interface-vpc-endpoints"></a>

To get started you do not need to change the settings for your streams, producers, or consumers\. Simply create an interface VPC endpoint in order for your Kinesis Data Streams traffic from and to your Amazon VPC resources to start flowing through the interface VPC endpoint\. 

The Kinesis Producer Library \(KPL\) and Kinesis Consumer Library \(KCL\) call AWS services like Amazon CloudWatch and Amazon DynamoDB using either public endpoints or private interface VPC endpoints, whichever are in use\. For example, if your KPL application is running in a VPC with DynamoDB interface VPC endpoints enabled, calls between DynamoDB and your KCL application flow through the interface VPC endpoint\.

## Availability<a name="availability"></a>

Interface VPC endpoints are currently supported within the following regions:
+ US East \(Ohio\)
+ US East \(N\. Virginia\)
+ US West \(Oregon\)
+ Asia Pacific \(Tokyo\)
+ EU \(Ireland\)
+ EU \(Paris\)