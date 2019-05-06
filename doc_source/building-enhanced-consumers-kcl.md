# Developing Consumers with Enhanced Fan\-Out Using the Kinesis Client Library 2\.0<a name="building-enhanced-consumers-kcl"></a>

Consumers that use *enhanced fan\-out* in Amazon Kinesis Data Streams can receive records from a data stream with dedicated throughput of up to 2 MiB of data per second per shard\. This type of consumer doesn't have to contend with other consumers that are receiving data from the stream\. For more information, see [Using Consumers with Enhanced Fan\-Out ](introduction-to-enhanced-consumers.md)\.

You can use version 2\.0 or later of the Kinesis Client Library \(KCL\) to develop applications that use enhanced fan\-out to receive data from streams\. The KCL automatically subscribes your application to all the shards of a stream, and ensures that your consumer application can read with a throughput value of 2 MiB/sec per shard\. If you want to use the KCL without turning on enhanced fan\-out, see [Developing Consumers Using the Kinesis Client Library 2\.0](https://integ-docs-aws.amazon.com/streams/latest/dev/developing-consumers-with-kcl-v2.html)\.

**Topics**
+ [Developing a Consumer Using the Kinesis Client Library 2\.x in Java](building-enhanced-consumers-kcl-java.md)