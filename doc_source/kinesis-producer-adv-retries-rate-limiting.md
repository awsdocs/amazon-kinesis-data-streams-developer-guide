# KPL Retries and Rate Limiting<a name="kinesis-producer-adv-retries-rate-limiting"></a>

When you add Kinesis Producer Library \(KPL\) user records using the KPL `addUserRecord()` operation, a record is given a time stamp and added to a buffer with a deadline set by the `RecordMaxBufferedTime` configuration parameter\. This time stamp/deadline combination sets the buffer priority\. Records are flushed from the buffer based on the following criteria:
+ Buffer priority
+ Aggregation configuration
+ Collection configuration

The aggregation and collection configuration parameters affecting buffer behavior are as follows:
+ `AggregationMaxCount`
+ `AggregationMaxSize`
+ `CollectionMaxCount`
+ `CollectionMaxSize`

Records flushed are then sent to your Kinesis data stream as Amazon Kinesis Data Streams records using a call to the Kinesis Data Streams API operation `PutRecords`\. The `PutRecords` operation sends requests to your stream that occasionally exhibit full or partial failures\. Records that fail are automatically added back to the KPL buffer\. The new deadline is set based on the minimum of these two values: 
+ Half the current `RecordMaxBufferedTime` configuration
+ The record’s time\-to\-live value

This strategy allows retried KPL user records to be included in subsequent Kinesis Data Streams API calls, to improve throughput and reduce complexity while enforcing the Kinesis Data Streams record’s time\-to\-live value\. There is no backoff algorithm, making this a relatively aggressive retry strategy\. Spamming due to excessive retries is prevented by rate limiting, discussed in the next section\.

## Rate Limiting<a name="kinesis-producer-adv-retries-rate-limiting-rate-limit"></a>

The KPL includes a rate limiting feature, which limits per\-shard throughput sent from a single producer\. Rate limiting is implemented using a token bucket algorithm with separate buckets for both Kinesis Data Streams records and bytes\. Each successful write to a Kinesis data stream adds a token \(or multiple tokens\) to each bucket, up to a certain threshold\. This threshold is configurable but by default is set 50 percent higher than the actual shard limit, to allow shard saturation from a single producer\. 

You can lower this limit to reduce spamming due to excessive retries\. However, the best practice is for each producer to retry for maximum throughput aggressively and to handle any resulting throttling determined as excessive by expanding the capacity of the stream and implementing an appropriate partition key strategy\.