# Reading Data from Amazon Kinesis Data Streams<a name="building-consumers"></a>

A *consumer* is an application that processes all data from a Kinesis data stream\. When a consumer uses *enhanced fan\-out*, it gets its own 2 MB/sec allotment of read throughput, allowing multiple consumers to read data from the same stream in parallel, without contending for read throughput with other consumers\. To use the enhanced fan\-out capability of shards, see [Developing Custom Consumers with Dedicated Throughput \(Enhanced Fan\-Out\)](enhanced-consumers.md)\.

 By default, shards in a stream provide 2 MB/sec of read throughput per shard\. This throughput gets shared across all the consumers that are reading from a given shard\. In other words, the default 2 MB/sec of throughput per shard is fixed, even if there are multiple consumers that are reading from the shard\. To use this default throughput of shards see, [Developing Custom Consumers with Shared Throughput](shared-throughput-consumers.md)\. 

The following table compares default throughput to enhanced fan\-out\. Message propagation delay is defined as the time taken in milliseconds for a payload sent using the payload\-dispatching APIs \(like PutRecord and PutRecords\) to reach the consumer application through the payload\-consuming APIs \(like GetRecords and SubscribeToShard\)\. 


| Characteristics | Unregistered Consumers without Enhanced Fan\-Out | Registered Consumers with Enhanced Fan\-Out | 
| --- | --- | --- | 
| Shard Read Throughput | Fixed at a total of 2 MB/sec per shard\. If there are multiple consumers reading from the same shard, they all share this throughput\. The sum of the throughputs they receive from the shard doesn't exceed 2 MB/sec\. | Scales as consumers register to use enhanced fan\-out\. Each consumer registered to use enhanced fan\-out receives its own read throughput per shard, up to 2 MB/sec, independently of other consumers\. | 
| Message propagation delay | An average of around 200 ms if you have one consumer reading from the stream\. This average goes up to around 1000 ms if you have five consumers\. | Typically an average of 70 ms whether you have one consumer or five consumers\. | 
| Cost | N/A | There is a data retrieval cost and a consumer\-shard hour cost\. For more information, see [Amazon Kinesis Data Streams Pricing](https://aws.amazon.com/kinesis/streams/pricing)\. | 
| Record delivery model | Pull model over HTTP using [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html)\. | Kinesis Data Streams pushes the records to you over HTTP/2 using [SubscribeToShard](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_SubscribeToShard.html)\. | 

**Topics**
+ [Developing Consumers Using AWS Lambda](lambda-consumer.md)
+ [Developing Consumers Using Amazon Kinesis Data Analytics](kda-consumer.md)
+ [Developing Consumers Using Amazon Kinesis Data Firehose](kdf-consumer.md)
+ [Using the Kinesis Client Library](shared-throughput-kcl-consumers.md)
+ [Developing Custom Consumers with Shared Throughput](shared-throughput-consumers.md)
+ [Developing Custom Consumers with Dedicated Throughput \(Enhanced Fan\-Out\)](enhanced-consumers.md)
+ [Migrating Consumers from KCL 1\.x to KCL 2\.x](kcl-migration.md)
+ [Troubleshooting Kinesis Data Streams Consumers](troubleshooting-consumers.md)
+ [Advanced Topics for Amazon Kinesis Data Streams Consumers](advanced-consumers.md)