# Writing Data to Amazon Kinesis Data Streams<a name="building-producers"></a>

A *producer* is an application that writes data to Amazon Kinesis Data Streams\. You can build producers for Kinesis Data Streams using the AWS SDK for Java and the Kinesis Producer Library\.

If you are new to Kinesis Data Streams, start by becoming familiar with the concepts and terminology presented in [What Is Amazon Kinesis Data Streams?](introduction.md) and [Getting Started with Amazon Kinesis Data Streams](getting-started.md)\.

**Important**  
Kinesis Data Streams supports changes to the data record retention period of your data stream\. For more information, see [Changing the Data Retention Period](kinesis-extended-retention.md)\.

To put data into the stream, you must specify the name of the stream, a partition key, and the data blob to be added to the stream\. The partition key is used to determine which shard in the stream the data record is added to\.

All the data in the shard is sent to the same worker that is processing the shard\. Which partition key you use depends on your application logic\. The number of partition keys should typically be much greater than the number of shards\. This is because the partition key is used to determine how to map a data record to a particular shard\. If you have enough partition keys, the data can be evenly distributed across the shards in a stream\.

**Topics**
+ [Developing Producers Using the Amazon Kinesis Producer Library](developing-producers-with-kpl.md)
+ [Developing Producers Using the Amazon Kinesis Data Streams API with the AWS SDK for Java](developing-producers-with-sdk.md)
+ [Writing to Amazon Kinesis Data Streams Using Kinesis Agent](writing-with-agents.md)
+ [Troubleshooting Amazon Kinesis Data Streams Producers](troubleshooting-producers.md)
+ [Advanced Topics for Kinesis Data Streams Producers](advanced-producers.md)