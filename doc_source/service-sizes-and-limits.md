# Amazon Kinesis Streams Limits<a name="service-sizes-and-limits"></a>

Kinesis Streams has following limits\.

+ The default shard limit is 500 shards for the following regions: US East \(N\. Virginia\), US West \(Oregon\), and EU \(Ireland\)\. For all other regions, the default shard limit is 200 shards\. There is no upper limit to the number of shards in a stream or account\. To view your limits and the number of shards in use, use the [DescribeLimits](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeLimits.html) API or the [describe\-limits](http://docs.aws.amazon.com/cli/latest/reference/kinesis/describe-limits.html) command\. To request an increase in your shard limit, use the [Kinesis Streams Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-kinesis)\.

+ [Changing the Data Retention Period](kinesis-extended-retention.md)

+ The maximum size of a data blob \(the data payload before base64\-encoding\) is up to 1 MB\.

+ [AddTagsToStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_AddTagsToStream.html), [ListTagsForStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListTagsForStream.html), and [RemoveTagsFromStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_RemoveTagsFromStream.html) can provide up to 5 transactions per second per account\.

+ [CreateStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_CreateStream.html), [DeleteStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DeleteStream.html), and [ListStreams](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListStreams.html) can provide up to 5 transactions per second\.

+ [DescribeStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeStream.html) can provide up to 10 transactions per second\.

+ [DescribeStreamSummary](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeStreamSummary.html) can provide up to 20 transactions per second\.

+ [GetRecords](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) can retrieve 10 MB of data\.

+ [DescribeLimits](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeLimits.html) can provide up to 1 transaction per second\.

+ [GetShardIterator](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html) can provide up to 5 transactions per second per open shard\.

+  [MergeShards](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_MergeShards.html) and [SplitShard](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_SplitShard.html) can provide up to 5 transactions per second\. 

+ [UpdateShardCount](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html) provides up to 2 calls per rolling 24\-hour period per region\. By default, the API can scale up to either your shard limit or 500 shards, whichever is lower\. To request an increase in the call rate limit, the shard limit for this API, or your overall shard limit, use the [Kinesis Streams Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-kinesis)\.

+ Each shard can support up to 5 transactions per second for reads, up to a maximum total data read rate of 2 MB per second\.

+ Each shard can support up to 1,000 records per second for writes, up to a maximum total data write rate of 1 MB per second \(including partition keys\)\. This write limit applies to operations such as [PutRecord](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html) and [PutRecords](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecords.html)\.

+ A shard iterator returned by [GetShardIterator](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html) times out after 5 minutes if you haven't used it\.

Read limits are based on the number of open shards\. For more information about shard states, see [Data Routing, Data Persistence, and Shard State after a Reshard](kinesis-using-sdk-java-after-resharding.md#kinesis-using-sdk-java-resharding-data-routing)\.

Many of these limits are directly related to API operations\. For more information, see the [Amazon Kinesis API Reference](http://docs.aws.amazon.com/kinesis/latest/APIReference/)\.