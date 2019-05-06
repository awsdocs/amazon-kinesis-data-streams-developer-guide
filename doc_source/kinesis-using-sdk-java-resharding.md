# Resharding a Stream<a name="kinesis-using-sdk-java-resharding"></a>

**Important**  
You can reshard your stream using the [UpdateShardCount](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html) API\. Otherwise, you can continue to perform splits and merges as explained here\.

Amazon Kinesis Data Streams supports *resharding*, which lets you adjust the number of shards in your stream to adapt to changes in the rate of data flow through the stream\. Resharding is considered an advanced operation\. If you are new to Kinesis Data Streams, return to this subject after you are familiar with all the other aspects of Kinesis Data Streams\.

There are two types of resharding operations: shard split and shard merge\. In a shard split, you divide a single shard into two shards\. In a shard merge, you combine two shards into a single shard\. Resharding is always *pairwise* in the sense that you cannot split into more than two shards in a single operation, and you cannot merge more than two shards in a single operation\. The shard or pair of shards that the resharding operation acts on are referred to as *parent* shards\. The shard or pair of shards that result from the resharding operation are referred to as *child* shards\. 

Splitting increases the number of shards in your stream and therefore increases the data capacity of the stream\. Because you are charged on a per\-shard basis, splitting increases the cost of your stream\. Similarly, merging reduces the number of shards in your stream and therefore decreases the data capacity—and cost—of the stream\. 

Resharding is typically performed by an administrative application that is distinct from the producer \(put\) applications and the consumer \(get\) applications\. Such an administrative application monitors the overall performance of the stream based on metrics provided by Amazon CloudWatch or based on metrics collected from the producers and consumers\. The administrative application also needs a broader set of IAM permissions than the consumers or producers because the consumers and producers usually should not need access to the APIs used for resharding\. For more information about IAM permissions for Kinesis Data Streams, see [Controlling Access to Amazon Kinesis Data Streams Resources Using IAM](controlling-access.md)\. 

**Topics**
+ [Strategies for Resharding](kinesis-using-sdk-java-resharding-strategies.md)
+ [Splitting a Shard](kinesis-using-sdk-java-resharding-split.md)
+ [Merging Two Shards](kinesis-using-sdk-java-resharding-merge.md)
+ [After Resharding](kinesis-using-sdk-java-after-resharding.md)