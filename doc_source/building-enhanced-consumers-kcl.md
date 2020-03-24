# Developing Enhanced Fan\-Out Consumers with KCL 2\.x<a name="building-enhanced-consumers-kcl"></a>

Consumers that use *enhanced fan\-out* in Amazon Kinesis Data Streams can receive records from a data stream with dedicated throughput of up to 2 MB of data per second per shard\. This type of consumer doesn't have to contend with other consumers that are receiving data from the stream\. For more information, see [Developing Custom Consumers with Dedicated Throughput \(Enhanced Fan\-Out\)](enhanced-consumers.md)\.

You can use version 2\.0 or later of the Kinesis Client Library \(KCL\) to develop applications that use enhanced fan\-out to receive data from streams\. The KCL automatically subscribes your application to all the shards of a stream, and ensures that your consumer application can read with a throughput value of 2 MB/sec per shard\. If you want to use the KCL without turning on enhanced fan\-out, see [Developing Consumers Using the Kinesis Client Library 2\.0](https://docs.aws.amazon.com/streams/latest/dev/developing-consumers-with-kcl-v2.html)\.

**Topics**
+ [Developing Enhanced Fan\-Out Consumers Using KCL 2\.x in Java](building-enhanced-consumers-kcl-java.md)