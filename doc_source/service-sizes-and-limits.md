# Kinesis Data Streams Quotas<a name="service-sizes-and-limits"></a>

Amazon Kinesis Data Streams has the following stream and shard quotas\.
+ There is no upper quota on the number of shards you can have in a stream or account\. It is common for a workload to have thousands of shards in a single stream\.
+ There is no upper quota on the number of streams you can have in an account\.
+ A single shard can ingest up to 1 MB of data per second \(including partition keys\) or 1,000 records per second for writes\. Similarly, if you scale your stream to 5,000 shards, the stream can ingest up to 5 GB per second or 5 million records per second\. If you need more ingest capacity, you can easily scale up the number of shards in the stream using the AWS Management Console or the [UpdateShardCount](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html) API\.
+ The default shard quota is 500 shards for the following AWS Regions: US East \(N\. Virginia\), US West \(Oregon\), and Europe \(Ireland\)\. For all other Regions, the default shard quota is 200 shards\.
+ The maximum size of the data payload of a record before base64\-encoding is up to 1 MB\.
+ [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) can retrieve up to 10 MB of data per call from a single shard, and up to 10,000 records per call\. Each call to `GetRecords` is counted as one read transaction\.
+ Each shard can support up to five read transactions per second\. Each read transaction can provide up to 10,000 records with an upper quota of 10 MB per transaction\.
+ Each shard can support up to a maximum total data read rate of 2 MB per second via [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html)\. If a call to `GetRecords` returns 10 MB, subsequent calls made within the next 5 seconds throw an exception\.

## API Quotas<a name="kds-api-limits"></a>

Like most AWS APIs, Kinesis Data Streams API operations are rate\-limited\. For information about API call rate quotas, see the [Amazon Kinesis API Reference](https://docs.aws.amazon.com/kinesis/latest/APIReference/)\. If you encounter API throttling, we encourage you to request a quota increase\.

## Increasing Quotas<a name="increasing-kds-limits"></a>

You can use Service Quotas to request an increase for a quota, if the quota is adjustable\. Some requests are automatically resolved, while others are submitted to AWS Support\. You can track the status of a quota increase request that is submitted to AWS Support\. Requests to increase service quotas do not receive priority support\. If you have an urgent request, contact AWS Support\. For more information, see [What Is Service Quotas?](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html)

To request a service quota increase, follow the procedure outlined in [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html)\.