# Monitoring Amazon Kinesis Data Streams<a name="monitoring"></a>

You can monitor your data streams in Amazon Kinesis Data Streams using the following features:
+ [CloudWatch metrics](monitoring-with-cloudwatch.md)— Kinesis Data Streams sends Amazon CloudWatch custom metrics with detailed monitoring for each stream\.
+ [Kinesis Agent](agent-health.md)— The Kinesis Agent publishes custom CloudWatch metrics to help assess if the agent is working as expected\.
+ [API logging](logging-using-cloudtrail.md)— Kinesis Data Streams uses AWS CloudTrail to log API calls and store the data in an Amazon S3 bucket\.
+ [Kinesis Client Library](monitoring-with-kcl.md)— Kinesis Client Library \(KCL\) provides metrics per shard, worker, and KCL application\.
+ [Kinesis Producer Library](monitoring-with-kpl.md)— Kinesis Producer Library \(KPL\) provides metrics per shard, worker, and KPL application\.