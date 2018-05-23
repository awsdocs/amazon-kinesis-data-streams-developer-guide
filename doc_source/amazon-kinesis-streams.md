# Kinesis Data Streams<a name="amazon-kinesis-streams"></a>

Amazon Kinesis Data Streams ingests a large amount of data in real time, durably stores the data, and makes the data available for consumption\. The unit of data stored by Kinesis Data Streams is a *data record*\. A *stream* represents a group of data records\. The data records in a stream are distributed into shards\.

A *shard* has a sequence of data records in a stream\. When you create a stream, you specify the number of shards for the stream\. Each shard can support up to 5 transactions per second for reads, up to a maximum total data read rate of 2 MB per second\. Shards also support up to 1,000 records per second for writes, up to a maximum total data write rate of 1 MB per second \(including partition keys\)\. The total capacity of a stream is the sum of the capacities of its shards\. You can increase or decrease the number of shards in a stream as needed\. However, you are charged on a per\-shard basis\.

A [producer](amazon-kinesis-producers.md) puts data records into shards and a [consumer](amazon-kinesis-consumers.md) gets data records from shards\.

## Determining the Initial Size of a Kinesis Data Stream<a name="how-do-i-size-a-stream"></a>

Before you create a stream, you need to determine an initial size for the stream\. After you create the stream, you can dynamically scale your shard capacity up or down using the AWS Management Console or the [UpdateShardCount](http://docs.aws.amazon.com/firehose/latest/APIReference/API_UpdateShardCount.html) API\. You can make updates while there is an Amazon Kinesis Data Streams application consuming data from the stream\.

To determine the initial size of a stream, you need the following input values:
+ The average size of the data record written to the stream in kilobytes \(KB\), rounded up to the nearest 1 KB, the data size \(`average_data_size_in_KB`\)\.
+ The number of data records written to and read from the stream per second \(`records_per_second`\)\.
+ The number of Amazon Kinesis Data Streams applications that consume data concurrently and independently from the stream, that is, the consumers \(`number_of_consumers`\)\.
+ The incoming write bandwidth in KB \(`incoming_write_bandwidth_in_KB`\), which is equal to the `average_data_size_in_KB` multiplied by the `records_per_second`\.
+ The outgoing read bandwidth in KB \(`outgoing_read_bandwidth_in_KB`\), which is equal to the `incoming_write_bandwidth_in_KB` multiplied by the `number_of_consumers`\.

You can calculate the initial number of shards \(`number_of_shards`\) that your stream will need by using the input values in the following formula:

```
number_of_shards = max(incoming_write_bandwidth_in_KB/1000, outgoing_read_bandwidth_in_KB/2000)
```

## Creating a Stream<a name="how-do-i-create-a-stream"></a>

You can create a stream using the Kinesis Data Streams console, the Kinesis Data Streams API, or the AWS CLI\.

**To create a stream using the console**

1. Open the Kinesis Data Streams console at [https://console\.aws\.amazon\.com/kinesis/](https://console.aws.amazon.com/kinesis/)\.

1. In the navigation bar, expand the region selector and choose a region\.

1. Choose **Create Stream**\.

1. On the **Create Stream** page, enter a name for your stream and the number of shards you need, and then click **Create**\.

   On the **Stream List** page, your stream's **Status** is `CREATING` while the stream is being created\. When the stream is ready to use, the **Status** changes to `ACTIVE`\.

1. Choose the name of your stream\. The **Stream Details** page displays a summary of your stream configuration, along with monitoring information\.

**To create a stream using the Kinesis Data Streams API**
+ For information about creating a stream using the Kinesis Data Streams API, see [Creating a Stream](kinesis-using-sdk-java-create-stream.md)\.

**To create a stream using the AWS CLI**
+ For information about creating a stream using the AWS CLI, see the [create\-stream](http://docs.aws.amazon.com/cli/latest/reference/kinesis/create-stream.html) command\.

## Updating a Stream<a name="updating-a-stream"></a>

You can update the details of a stream using the Kinesis Data Streams console, the Kinesis Data Streams API, or the AWS CLI\.

**Note**  
You can enable server\-side encryption for existing streams, or for streams that you have recently created\.

### <a name="update-stream-console"></a>

**To update a stream using the console**

1. Open the Kinesis Data Streams console at [https://console\.aws\.amazon\.com/kinesis/](https://console.aws.amazon.com/kinesis/)\.

1. In the navigation bar, expand the region selector and select a region\.

1. Choose the name of your stream\. The **Stream Details** page displays a summary of your stream configuration and monitoring information\.

1. To edit the number of shards, choose **Edit** in the **Shards** section, and then enter a new shard count\.

1. To enable server\-side encryption of data records, choose **Edit** in the **Server\-side encryption** section\. Choose a KMS key to use as the master key for encryption, or use the default master key, **aws/kinesis**, managed by Kinesis\. If you enable encryption for a stream and use your own KMS master key, ensure that your producer and consumer applications have access to the KMS master key you used\. To assign permissions to an application to access a user\-generated KMS key, see [Permissions to Use User\-Generated KMS Master Keys](permissions-user-key-KMS.md)

1. To edit the data retention period, choose **Edit** in the **Data retention period** section, and then enter a new data retention period\.

1. If you have enabled custom metrics on your account, choose **Edit** in the **Shard level metrics** section, and then specify metrics for your stream\. For more information, see [Monitoring the Amazon Kinesis Data Streams Service with Amazon CloudWatch](monitoring-with-cloudwatch.md)\.

### Updating a Stream Using the API<a name="update-stream-api"></a>

To update stream details using the API, see the following methods:
+ [AddTagsToStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_AddTagsToStream.html)
+ [DecreaseStreamRetentionPeriod](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DecreaseStreamRetentionPeriod.html)
+ [DisableEnhancedMonitoring](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DisableEnhancedMonitoring.html)
+ [EnableEnhancedMonitoring](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_EnableEnhancedMonitoring.html)
+ [IncreaseStreamRetentionPeriod](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_IncreaseStreamRetentionPeriod.html)
+ [RemoveTagsFromStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_RemoveTagsFromStream.html)
+ [StartStreamEncryption](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_StartStreamEncryption.html)
+ [StopStreamEncryption](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_StopStreamEncryption.html)
+ [UpdateShardCount](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html)

### Updating a Stream Using the CLI<a name="update-stream-cli"></a>

For information about updating a stream using the AWS CLI, see the [Kinesis CLI reference](http://docs.aws.amazon.com/cli/latest/reference/kinesis/index.html)\. 