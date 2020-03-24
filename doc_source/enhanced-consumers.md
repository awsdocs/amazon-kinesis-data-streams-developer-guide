# Developing Custom Consumers with Dedicated Throughput \(Enhanced Fan\-Out\)<a name="enhanced-consumers"></a>

In Amazon Kinesis Data Streams, you can build consumers that use a feature called *enhanced fan\-out*\. This feature enables consumers to receive records from a stream with throughput of up to 2 MB of data per second per shard\. This throughput is dedicated, which means that consumers that use enhanced fan\-out don't have to contend with other consumers that are receiving data from the stream\. Kinesis Data Streams pushes data records from the stream to consumers that use enhanced fan\-out\. Therefore, these consumers don't need to poll for data\.

**Important**  
You can register up to twenty consumers per stream to use enhanced fan\-out\. 

The following diagram shows the enhanced fan\-out architecture\. If you use version 2\.0 or later of the Amazon Kinesis Client Library \(KCL\) to build a consumer, the KCL sets up the consumer to use enhanced fan\-out to receive data from all the shards of the stream\. If you use the API to build a consumer that uses enhanced fan\-out, then you can subscribe to individual shards\.

![\[Workflow diagram showing enhanced fan-out architecture with two shards and two consumers. Each of the two consumers is using enhanced fan-out to receive data from both shards of the stream.\]](http://docs.aws.amazon.com/streams/latest/dev/images/enhanced_fan-out.png)

The diagram shows the following: 
+ A stream with two shards\.
+ Two consumers that are using enhanced fan\-out to receive data from the stream: Consumer X and Consumer Y\. Each of the two consumers is subscribed to all of the shards and all of the records of the stream\. If you use version 2\.0 or later of the KCL to build a consumer, the KCL automatically subscribes that consumer to all the shards of the stream\. On the other hand, if you use the API to build a consumer, you can subscribe to individual shards\. 
+ Arrows representing the enhanced fan\-out pipes that the consumers use to receive data from the stream\. An enhanced fan\-out pipe provides up to 2 MB/sec of data per shard, independently of any other pipes or of the total number of consumers\.

**Topics**
+ [Developing Enhanced Fan\-Out Consumers with KCL 2\.x](building-enhanced-consumers-kcl.md)
+ [Developing Enhanced Fan\-Out Consumers with the Kinesis Data Streams API](building-enhanced-consumers-api.md)
+ [Managing Enhanced Fan\-Out Consumers with the AWS Management Console](building-enhanced-consumers-console.md)