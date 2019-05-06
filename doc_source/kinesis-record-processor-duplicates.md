# Handling Duplicate Records<a name="kinesis-record-processor-duplicates"></a>

There are two primary reasons why records may be delivered more than one time to your Amazon Kinesis Data Streams application: producer retries and consumer retries\. Your application must anticipate and appropriately handle processing individual records multiple times\.

## Producer Retries<a name="kinesis-record-processor-duplicates-producer"></a>

Consider a producer that experiences a network\-related timeout after it makes a call to `PutRecord`, but before it can receive an acknowledgement from Amazon Kinesis Data Streams\. The producer cannot be sure if the record was delivered to Kinesis Data Streams\. Assuming that every record is important to the application, the producer would have been written to retry the call with the same data\. If both `PutRecord` calls on that same data were successfully committed to Kinesis Data Streams, then there will be two Kinesis Data Streams records\. Although the two records have identical data, they also have unique sequence numbers\. Applications that need strict guarantees should embed a primary key within the record to remove duplicates later when processing\. Note that the number of duplicates due to producer retries is usually low compared to the number of duplicates due to consumer retries\.

**Note**  
If you use the AWS SDK `PutRecord`, the default [configuration retries a failed `PutRecord` call up to three times\.](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/ClientConfiguration.html)

## Consumer Retries<a name="kinesis-record-processor-duplicates-consumer"></a>

Consumer \(data processing application\) retries happen when record processors restart\. Record processors for the same shard restart in the following cases:

1. A worker terminates unexpectedly 

1. Worker instances are added or removed 

1. Shards are merged or split 

1. The application is deployed 

In all these cases, the shards\-to\-worker\-to\-record\-processor mapping is continuously updated to load balance processing\. Shard processors that were migrated to other instances restart processing records from the last checkpoint\. This results in duplicated record processing as shown in the example below\. For more information about load\-balancing, see [Resharding, Scaling, and Parallel Processing](kinesis-record-processor-scaling.md)\.

### Example: Consumer Retries Resulting in Redelivered Records<a name="kinesis-record-processor-duplicates-consumer-example"></a>

In this example, you have an application that continuously reads records from a stream, aggregates records into a local file, and uploads the file to Amazon S3\. For simplicity, assume there is only 1 shard and 1 worker processing the shard\. Consider the following example sequence of events, assuming that the last checkpoint was at record number 10000:

1.  A worker reads the next batch of records from the shard, records 10001 to 20000\.

1.  The worker then passes the batch of records to the associated record processor\.

1.  The record processor aggregates the data, creates an Amazon S3 file, and uploads the file to Amazon S3 successfully\.

1.  Worker terminates unexpectedly before a new checkpoint can occur\. 

1.  Application, worker, and record processor restart\.

1.  Worker now begins reading from the last successful checkpoint, in this case 10001\.

Thus, records 10001\-20000 are consumed more than one time\.

### Being Resilient to Consumer Retries<a name="kinesis-record-processor-duplicates-consumer-resilience"></a>

Even though records may be processed more than one time, your application may want to present the side effects as if records were processed only one time \(idempotent processing\)\. Solutions to this problem vary in complexity and accuracy\. If the destination of the final data can handle duplicates well, we recommend relying on the final destination to achieve idempotent processing\. For example, with [Elasticsearch](http://www.elasticsearch.org/) you can use a combination of versioning and unique IDs to prevent duplicated processing\. 

In the example application in the previous section, it continuously reads records from a stream, aggregates records into a local file, and uploads the file to Amazon S3\. As illustrated, records 10001 \-20000 are consumed more than one time resulting in multiple Amazon S3 files with the same data\. One way to mitigate duplicates from this example is to ensure that step 3 uses the following scheme: 

1.  Record Processor uses a fixed number of records per Amazon S3 file, such as 5000\.

1.  The file name uses this schema: Amazon S3 prefix, shard\-id, and `First-Sequence-Num`\. In this case, it could be something like `sample-shard000001-10001`\.

1.  After you upload the Amazon S3 file, checkpoint by specifying `Last-Sequence-Num`\. In this case, you would checkpoint at record number 15000\. 

With this scheme, even if records are processed more than one time, the resulting Amazon S3 file has the same name and has the same data\. The retries only result in writing the same data to the same file more than one time\.

In the case of a reshard operation, the number of records left in the shard may be less than your desired fixed number needed\. In this case, your `shutdown()` method has to flush the file to Amazon S3 and checkpoint on the last sequence number\. The above scheme is compatible with reshard operations as well\.