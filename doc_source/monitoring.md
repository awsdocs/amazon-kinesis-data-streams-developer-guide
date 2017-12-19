# Monitoring Amazon Kinesis Streams<a name="monitoring"></a>

You can monitor Amazon Kinesis Streams using the following features:

+ CloudWatch metrics— Kinesis Streams sends Amazon CloudWatch custom metrics with detailed monitoring for each stream\.

+ Kinesis Agent— The Kinesis Agent publishes custom CloudWatch metrics to help assess if the agent is working as expected\.

+ API logging— Kinesis Streams uses AWS CloudTrail to log API calls and store the data in an Amazon S3 bucket\.

+ Kinesis Client Library— Kinesis Streams Client Library \(KCL\) provides metrics per shard, worker, and KCL application\.

+ Kinesis Producer Library— Kinesis Streams Producer Library \(KPL\) provides metrics per shard, worker, and KPL application\.