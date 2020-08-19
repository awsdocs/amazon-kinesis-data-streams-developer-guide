# Resharding, Scaling, and Parallel Processing<a name="kinesis-record-processor-scaling"></a>

*Resharding* enables you to increase or decrease the number of shards in a stream in order to adapt to changes in the rate of data flowing through the stream\. Resharding is typically performed by an administrative application that monitors shard data\-handling metrics\. Although the KCL itself doesn't initiate resharding operations, it is designed to adapt to changes in the number of shards that result from resharding\. 

As noted in [Using a Lease Table to Track the Shards Processed by the KCL Consumer Application](shared-throughput-kcl-consumers.md#shared-throughput-kcl-consumers-leasetable), the KCL tracks the shards in the stream using an Amazon DynamoDB table\. When new shards are created as a result of resharding, the KCL discovers the new shards and populates new rows in the table\. The workers automatically discover the new shards and create processors to handle the data from them\. The KCL also distributes the shards in the stream across all the available workers and record processors\. 

The KCL ensures that any data that existed in shards prior to the resharding is processed first\. After that data has been processed, data from the new shards is sent to record processors\. In this way, the KCL preserves the order in which data records were added to the stream for a particular partition key\.

## Example: Resharding, Scaling, and Parallel Processing<a name="kinesis-record-processor-scaling-example"></a>

The following example illustrates how the KCL helps you handle scaling and resharding:
+ For example, if your application is running on one EC2 instance, and is processing one Kinesis data stream that has four shards\. This one instance has one KCL worker and four record processors \(one record processor for every shard\)\. These four record processors run in parallel within the same process\. 
+ Next, if you scale the application to use another instance, you have two instances processing one stream that has four shards\. When the KCL worker starts up on the second instance, it load\-balances with the first instance, so that each instance now processes two shards\. 
+ If you then decide to split the four shards into five shards\. The KCL again coordinates the processing across instances: one instance processes three shards, and the other processes two shards\. A similar coordination occurs when you merge shards\.

Typically, when you use the KCL, you should ensure that the number of instances does not exceed the number of shards \(except for failure standby purposes\)\. Each shard is processed by exactly one KCL worker and has exactly one corresponding record processor, so you never need multiple instances to process one shard\. However, one worker can process any number of shards, so it's fine if the number of shards exceeds the number of instances\. 

To scale up processing in your application, you should test a combination of these approaches:
+ Increasing the instance size \(because all record processors run in parallel within a process\)
+ Increasing the number of instances up to the maximum number of open shards \(because shards can be processed independently\)
+ Increasing the number of shards \(which increases the level of parallelism\)

Note that you can use Auto Scaling to automatically scale your instances based on appropriate metrics\. For more information, see the [Amazon EC2 Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)\.

When resharding increases the number of shards in the stream, the corresponding increase in the number of record processors increases the load on the EC2 instances that are hosting them\. If the instances are part of an Auto Scaling group, and the load increases sufficiently, the Auto Scaling group adds more instances to handle the increased load\. You should configure your instances to launch your Amazon Kinesis Data Streams application at startup, so that additional workers and record processors become active on the new instance right away\.

For more information about resharding, see [Resharding a Stream](kinesis-using-sdk-java-resharding.md)\. 