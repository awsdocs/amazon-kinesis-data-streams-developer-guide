# Using Amazon Kinesis Data Streams with Interface VPC Endpoints<a name="vpc"></a>

## Interface VPC endpoints for Kinesis Data Streams<a name="interface-vpc-endpoints"></a>

You can use an interface VPC endpoint to keep traffic between your Amazon VPC and Kinesis Data Streams from leaving the Amazon network\. Interface VPC endpoints don't require an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection\. Interface VPC endpoints are powered by AWS PrivateLink, an AWS technology that enables private communication between AWS services using an elastic network interface with private IPs in your Amazon VPC\. For more information, see [Amazon Virtual Private Cloud](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html)\. 

## Using interface VPC endpoints for Kinesis Data Streams<a name="using-interface-vpc-endpoints"></a>

To get started you do not need to change the settings for your streams, producers, or consumers\. Simply create an interface VPC endpoint in order for your Kinesis Data Streams traffic from and to your Amazon VPC resources to start flowing through the interface VPC endpoint\. 

The Kinesis Producer Library \(KPL\) and Kinesis Consumer Library \(KCL\) call AWS services like Amazon CloudWatch and Amazon DynamoDB\. These services are reached using either public service endpoints or private VPC endpoints, depending on the configuration of the VPC containing your application\. For example, if your KPL application is running in a VPC with a DynamoDB gateway VPC endpoint configured, calls between your KPL / KCL application and DynamoDB would flow through the gateway VPC endpoint\. Similarly, if your VPC is configured to use an AWS CloudWatch interface VPC endpoint, calls between your KPL / KCL application and AWS Cloudwatch would flow through this interface VPC endpoint\. 

## Availability<a name="availability"></a>

Interface VPC endpoints are currently supported within the following Regions:
+ US West \(Oregon\)
+ EU \(Paris\)
+ US East \(N\. Virginia\)
+ EU \(Ireland\)
+ Asia Pacific \(Mumbai\)
+ US East \(Ohio\)
+ EU \(Frankfurt\)
+ South America \(São Paulo\)
+ Asia Pacific \(Seoul\)
+ EU \(London\)
+ Asia Pacific \(Tokyo\)
+ US West \(N\. California\)
+ Asia Pacific \(Singapore\)
+ Asia Pacific \(Sydney\)
+ Canada \(Central\)