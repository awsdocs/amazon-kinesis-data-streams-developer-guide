# Tracking Amazon Kinesis Data Streams Application State<a name="kinesis-record-processor-ddb"></a>

For each Amazon Kinesis Data Streams application, the KCL uses a unique Amazon DynamoDB table to keep track of the application's state\. Because the KCL uses the name of the Amazon Kinesis Data Streams application to create the name of the table, each application name must be unique\.

You can view the table using the [Amazon DynamoDB console](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ConsoleDynamoDB.html) while the application is running\.

If the Amazon DynamoDB table for your Amazon Kinesis Data Streams application does not exist when the application starts up, one of the workers creates the table and calls the `describeStream` method to populate the table\. For more information, see [Application State Data](#kinesis-record-processor-ddb-table-contents)\.

**Important**  
 Your account is charged for the costs associated with the DynamoDB table, in addition to the costs associated with Kinesis Data Streams itself\. 

## Throughput<a name="kinesis-record-processor-ddb-throughput"></a>

If your Amazon Kinesis Data Streams application receives provisioned\-throughput exceptions, you should increase the provisioned throughput for the DynamoDB table\. The KCL creates the table with a provisioned throughput of 10 reads per second and 10 writes per second, but this might not be sufficient for your application\. For example, if your Amazon Kinesis Data Streams application does frequent checkpointing or operates on a stream that is composed of many shards, you might need more throughput\.

For information about provisioned throughput in DynamoDB, see [Read/Write Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html) and [Working with Tables and Data](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithDDTables.html) in the *Amazon DynamoDB Developer Guide*\.

## Application State Data<a name="kinesis-record-processor-ddb-table-contents"></a>

Each row in the DynamoDB table represents a shard that is being processed by your application\. The hash key for the table is **leaseKey**, which is the shard ID\.

In addition to the shard ID, each row also includes the following data:
+ **checkpoint:** The most recent checkpoint sequence number for the shard\. This value is unique across all shards in the stream\.
+ **checkpointSubSequenceNumber:** When using the Kinesis Producer Library's aggregation feature, this is an extension to **checkpoint** that tracks individual user records within the Kinesis record\.
+ **leaseCounter:** Used for lease versioning so that workers can detect that their lease has been taken by another worker\.
+ **leaseKey:** A unique identifier for a lease\. Each lease is particular to a shard in the stream and is held by one worker at a time\.
+ **leaseOwner:** The worker that is holding this lease\.
+ **ownerSwitchesSinceCheckpoint:** How many times this lease has changed workers since the last time a checkpoint was written\.
+ **parentShardId:** Used to ensure that the parent shard is fully processed before processing starts on the child shards\. This ensures that records are processed in the same order they were put into the stream\.