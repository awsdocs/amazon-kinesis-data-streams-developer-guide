# Costs, Regions, and Performance Considerations<a name="costs-performance"></a>

When you apply server\-side encryption, you are subject to AWS KMS API usage and key costs\. Unlike custom KMS master keys, the `(Default) aws/kinesis` customer master key \(CMK\) is offered free of charge\. However, you still must pay for the API usage costs that Amazon Kinesis Data Streams incurs on your behalf\.

API usage costs apply for every CMK, including custom ones\. Kinesis Data Streams calls AWS KMS approximately every five minutes when it is rotating the data key\. In a 30\-day month, the total cost of AWS KMS API calls that are initiated by a Kinesis stream should be less than a few dollars\. This cost scales with the number of user credentials that you use on your data producers and consumers because each user credential requires a unique API call to AWS KMS\. When you use an IAM role for authentication, each assume role call results in unique user credentials\. To save KMS costs, you might want to cache user credentials that are returned by the assume role call\. 

The following describes the costs by resource:

**Keys**
+ The CMK for Kinesis that's managed by AWS \(alias = `aws/kinesis`\) is free\.
+ User\-generated KMS keys are subject to KMS key costs\. For more information, see [AWS Key Management Service Pricing](http://aws.amazon.com/kms/pricing/#Keys)\.

## KMS API Usage<a name="api-usage"></a>

For every encrypted stream, the Kinesis service calls the AWS KMS service approximately every five minutes to create a new data encryption key\. In a 30\-day month, each encrypted stream generates approximately 8,640 KMS API requests\. API requests to generate new data encryption keys are subject to AWS KMS usage costs\. For more information, see [AWS Key Management Service Pricing: Usage](http://aws.amazon.com/kms/pricing/#Usage)\.

## Availability of Server\-Side Encryption by Region<a name="sse-regions"></a>

Server\-side encryption of Kinesis streams is available in the following regions\.


| Region Name | Region | 
| --- | --- | 
| US East \(Ohio\) | us\-east\-2 | 
| US East \(N\. Virginia\) | us\-east\-1 | 
| US West \(Oregon\) | us\-west\-2 | 
| US West \(N\. California\) | us\-west\-1 | 
| AWS GovCloud \(US\-West\) | us\-gov\-west\-1 | 
| Canada \(Central\) | ca\-central\-1 | 
| Europe \(Ireland\) | eu\-west\-1 | 
| Europe \(London\) | eu\-west\-2 | 
| Europe \(Frankfurt\) | eu\-central\-1 | 
| Asia Pacific \(Tokyo\) Region | ap\-northeast\-1 | 
| Asia Pacific \(Seoul\) Region | ap\-northeast\-2 | 
| Asia Pacific \(Singapore\) | ap\-southeast\-1 | 
| Asia Pacific \(Mumbai\) | ap\-south\-1 | 
| Asia Pacific \(Sydney\) | ap\-southeast\-2 | 
| South America \(São Paulo\) | sa\-east\-1 | 

## Performance Considerations<a name="performance-considerations"></a>

Due to the service overhead of applying encryption, applying server\-side encryption increases the typical latency of `PutRecord`, `PutRecords`, and `GetRecords` by less than 100μs\.