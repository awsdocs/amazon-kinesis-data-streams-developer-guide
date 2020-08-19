# Kinesis Data Streams Quotas and Limits<a name="service-sizes-and-limits"></a>

Amazon Kinesis Data Streams has the following stream and shard quotas and limits\.
+ There is no upper quota on the number of streams you can have in an account\.
+ The default shard quota is 500 shards per data stream for the following AWS regions: US East \(N\. Virginia\), US West \(Oregon\), and Europe \(Ireland\)\. For all other regions, the default shard quota is 200 shards per data stream\. 

  To request the shards per data stream quota increase, follow the procedure outlined in [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html)\.
+ A single shard can ingest up to 1 MB of data per second \(including partition keys\) or 1,000 records per second for writes\. Similarly, if you scale your stream to 5,000 shards, the stream can ingest up to 5 GB per second or 5 million records per second\. If you need more ingest capacity, you can easily scale up the number of shards in the stream using the AWS Management Console or the [UpdateShardCount](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html) API\.
+ The maximum size of the data payload of a record before base64\-encoding is up to 1 MB\.
+ [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) can retrieve up to 10 MB of data per call from a single shard, and up to 10,000 records per call\. Each call to `GetRecords` is counted as one read transaction\.
+ Each shard can support up to five read transactions per second\. Each read transaction can provide up to 10,000 records with an upper quota of 10 MB per transaction\.
+ Each shard can support up to a maximum total data read rate of 2 MB per second via [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html)\. If a call to `GetRecords` returns 10 MB, subsequent calls made within the next 5 seconds throw an exception\.

## API Limits<a name="kds-api-limits"></a>

Like most AWS APIs, Kinesis Data Streams API operations are rate\-limited\. The following limits apply per AWS account per region\. For more information on Kinesis Data Streams APIs, see the [Amazon Kinesis API Reference](https://docs.aws.amazon.com/kinesis/latest/APIReference/)\. 

### KDS Control Plane API Limits<a name="kds-api-limits-control"></a>

The following section describes limits for the KDS control plane APIs\. KDS control plane APIs enable you to create and manage your data streams\. These limits apply per AWS account per region\.


**Control Plane API Limits**  

| API | API call limit | Stream\-level limit | Additional details | 
| --- | --- | --- | --- | 
| AddTagsToStream | 5 transactions per second \(TPS\) | 50 tags per data stream per account per region  |  | 
| CreateStream | 5 TPS | There is no upper quota on the number of streams you can have in an account\.  | You receive a LimitExceededException when making a CreateStream request when you try to do one of the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html) | 
| DecreaseStreamRetentionPeriod | 5 TPS | The minimum value of a data stream's retention period is 24 hours\.  |  | 
| DeleteStream | 5 TPS | N/A |  | 
| DeregisterStreamConsumer | 5 TPS | N/A |  | 
| DescribeLimits | 1 TPS |  |  | 
| DescribeStream | 10 TPS | N/A |  | 
| DescribeStreamConsumer | 20 TPS | N/A |  | 
| DescribeStreamSummary | 20 TPS | N/A |  | 
| DisableEnhancedMonitoring | 5 TPS | N/A |  | 
| EnableEnhancedMonitoring | 5 TPS | N/A |  | 
| IncreaseStreamRetentionPeriod | 5 TPS | The maximum value of a stream's retention period is 168 hours \(7 days\)\.  |  | 
| ListShards | 100 TPS | N/A |  | 
| ListStreamConsumers | 5 TPS | N/A |  | 
| ListStreams | 5 TPS | N/A |  | 
| ListTagsForStream | 5 TPS | N/A |  | 
| MergeShards | 5 TPS | N/A |  | 
| RegisterStreamConsumer | 5 TPS | You can register up to 20 consumers per data stream\. A given consumer can only be registered with one data stream at a time\. Only 5 consumers can be created simultaneously\. In other words, you cannot have more than 5 consumers in a CREATING status at the same time\. Registering a 6th consumer while there are 5 in a CREATING status results in a LimitExceededException\.  |  | 
| RemoveTagsFromStream | 5 TPS | N/A |  | 
| SplitShard | 5 TPS | N/A |  | 
| StartStreamEncryption |  |  | You can successfully apply a new AWS KMS key for server\-side encryption 25 times in a rolling 24\-hour period\.  | 
| StopStreamEncryption |  |  | You can successfully disable server\-side encryption 25 times in a rolling 24\-hour period\.  | 
| UpdateShardCount |  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)  |  | 

### KDS Data Plane API Limits<a name="kds-api-limits-data"></a>

The following section describes the limits for the KDS data plane APIs\. KDS data plane APIs enable you to use your data streams for collecting and processing data records in real time\. These limits apply per shard within your data streams\.


**Data Plane API limits**  

| API | API call limit | Payload limit | Additional details | 
| --- | --- | --- | --- | 
| GetRecords | 5 TPS | The maximum number of records that can be returned per call is 10,000\. The maximum size of data that GetRecords can return is 10 MB\.  | If a call returns this amount of data, subsequent calls made within the next 5 seconds throw ProvisionedThroughputExceededException\. If there is insufficient provisioned throughput on the stream, subsequent calls made within the next 1 second throw ProvisionedThroughputExceededException\. | 
| GetShardIterator | 5 TPS |  | A shard iterator expires 5 minutes after it is returned to the requester\. If a GetShardIterator request is made too often, you receive a ProvisionedThroughputExceededException\. | 
| PutRecord | 1000 TPS | Each shard can support writes up to 1,000 records per second, up to a maximum data write total of 1 MB per second\.  |  | 
| PutRecords |  | Each PutRecords request can support up to 500 records\. Each record in the request can be as large as 1 MB, up to a limit of 5 MB for the entire request, including partition keys\. Each shard can support writes up to 1,000 records per second, up to a maximum data write total of 1 MB per second\.  |  | 
| SubscribeToShard | You can make one call to SubscribeToShard per second per registered consumer per shard\.  |  | If you call SubscribeToShard again with the same ConsumerARN and ShardId within 5 seconds of a successful call, you'll get a ResourceInUseException\.  | 

## Increasing Quotas<a name="increasing-kds-limits"></a>

You can use Service Quotas to request an increase for a quota, if the quota is adjustable\. Some requests are automatically resolved, while others are submitted to AWS Support\. You can track the status of a quota increase request that is submitted to AWS Support\. Requests to increase service quotas do not receive priority support\. If you have an urgent request, contact AWS Support\. For more information, see [What Is Service Quotas?](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html)

To request a service quota increase, follow the procedure outlined in [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html)\.