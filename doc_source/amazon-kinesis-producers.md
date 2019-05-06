# Kinesis Data Streams Producers<a name="amazon-kinesis-producers"></a>

A *producer* puts data records into Amazon Kinesis data streams\. For example, a web server sending log data to a Kinesis data stream is a producer\. A [consumer](amazon-kinesis-consumers.md) processes the data records from a stream\.

**Important**  
Kinesis Data Streams supports changes to the data record retention period of your data stream\. For more information, see [Changing the Data Retention Period](kinesis-extended-retention.md)\.

To put data into the stream, you must specify the name of the stream, a partition key, and the data blob to be added to the stream\. The partition key is used to determine which shard in the stream the data record is added to\.

All the data in the shard is sent to the same worker that is processing the shard\. Which partition key you use depends on your application logic\. The number of partition keys should typically be much greater than the number of shards\. This is because the partition key is used to determine how to map a data record to a particular shard\. If you have enough partition keys, the data can be evenly distributed across the shards in a stream\.

For more information, see [Adding Data to a Stream](developing-producers-with-sdk.md#kinesis-using-sdk-java-add-data-to-stream) \(includes Java example code\), the [PutRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecords.html) and [PutRecord](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html) operations in the Kinesis Data Streams API, or the [put\-record](https://docs.aws.amazon.com/cli/latest/reference/kinesis/put-record.html) command\.