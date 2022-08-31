# Creating and Managing Streams<a name="working-with-streams"></a>

Amazon Kinesis Data Streams ingests a large amount of data in real time, durably stores the data, and makes the data available for consumption\. The unit of data stored by Kinesis Data Streams is a *data record*\. A *data stream* represents a group of data records\. The data records in a data stream are distributed into shards\.

A *shard* has a sequence of data records in a stream\. It serves as a base throughput unit of a Kinesis data stream\. A shard supports 1 MB/s and 1000 records per second for *writes* and 2 MB/s for *reads* in both on\-demand and provisioned capacity modes\. The shard limits ensure predictable performance, making it easier to design and operate a highly reliable data streaming workflow\. 

**Topics**
+ [Choosing the Data Stream Capacity Mode](how-do-i-size-a-stream.md)
+ [Creating a Stream via the AWS Management Console](how-do-i-create-a-stream.md)
+ [Creating a Stream via the APIs](kinesis-using-sdk-java-create-stream.md)
+ [Updating a Stream](updating-a-stream.md)
+ [Listing Streams](kinesis-using-sdk-java-list-streams.md)
+ [Listing Shards](kinesis-using-sdk-java-list-shards.md)
+ [Deleting a Stream](kinesis-using-sdk-java-delete-stream.md)
+ [Resharding a Stream](kinesis-using-sdk-java-resharding.md)
+ [Changing the Data Retention Period](kinesis-extended-retention.md)
+ [Tagging Your Streams in Amazon Kinesis Data Streams](tagging.md)