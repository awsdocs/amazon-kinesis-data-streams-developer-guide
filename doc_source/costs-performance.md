# Costs, Regions, and Performance Considerations<a name="costs-performance"></a>

When you apply server\-side encryption, you are subject to AWS KMS API usage and key costs\. Unlike custom KMS master keys, the `(Default) aws/kinesis` customer master key \(CMK\) is offered free of charge\. However, you still must pay for the API usage costs that Amazon Kinesis Data Streams incurs on your behalf\.

API usage costs apply for every CMK, including custom ones\. Kinesis Data Streams calls AWS KMS approximately every five minutes when it is rotating the data key\. In a 30\-day month, the total cost of AWS KMS API calls that are initiated by a Kinesis stream should be less than a few dollars\. This cost scales with the number of user credentials that you use on your data producers and consumers because each user credential requires a unique API call to AWS KMS\. When you use an IAM role for authentication, each assume role call results in unique user credentials\. To save KMS costs, you might want to cache user credentials that are returned by the assume role call\. 

The following describes the costs by resource:

**Keys**
+ The CMK for Kinesis that's managed by AWS \(alias = `aws/kinesis`\) is free\.
+ User\-generated KMS keys are subject to KMS key costs\. For more information, see [AWS Key Management Service Pricing](http://aws.amazon.com/kms/pricing/#Keys)\.

API usage costs apply for every CMK, including custom ones\. Kinesis Data Streams calls KMS approximately every 5 minutes when it is rotating the data key\. In a 30\-day month, the total cost of KMS API calls initiated by a Kinesis data stream should be less than a few dollars\. Please note that this cost scales with the number of user credentials you use on your data producers and consumers because each user credential requires a unique API call to AWS KMS\. When you use IAM role for authentication, each assume\-role\-call will result in unique user credentials and you might want to cache user credentials returned by the assume\-role\-call to save KMS costs\.

## KMS API Usage<a name="api-usage"></a>

For every encrypted stream, when reading from TIP and using a single IAM account/user access key across readers and writers, Kinesis service calls the AWS KMS service approximately 12 times every 5 minutes\. Not reading from TIP could lead to higher calls to AWS KMS service\. API requests to generate new data encryption keys are subject to AWS KMS usage costs\. For more information, see [AWS Key Management Service Pricing: Usage](http://aws.amazon.com/kms/pricing/#Usage)\.

## Availability of Server\-Side Encryption by Region<a name="sse-regions"></a>

Currently, server\-side encryption of Kinesis streams is available in all the regions supported for Kinesis Data Streams, including AWS GovCloud \(US\-West\), and the China regions\. For more information about supported regions for Kinesis Data Streams see [https://docs\.aws\.amazon\.com/general/latest/gr/ak\.html](https://docs.aws.amazon.com/general/latest/gr/ak.html)\.

## Performance Considerations<a name="performance-considerations"></a>

Due to the service overhead of applying encryption, applying server\-side encryption increases the typical latency of `PutRecord`, `PutRecords`, and `GetRecords` by less than 100μs\.