# Troubleshooting Kinesis Data Streams Consumers<a name="troubleshooting-consumers"></a>

**Topics**
+ [Some Kinesis Data Streams Records are Skipped When Using the Kinesis Client Library](#w7aac19c27b5)
+ [Records Belonging to the Same Shard are Processed by Different Record Processors at the Same Time](#records-belonging-to-the-same-shard)
+ [Consumer Application is Reading at a Slower Rate Than Expected](#consumer-app-reading-slower)
+ [GetRecords Returns Empty Records Array Even When There is Data in the Stream](#getrecords-returns-empty)
+ [Shard Iterator Expires Unexpectedly](#shard-iterator-expires-unexpectedly)
+ [Consumer Record Processing Falling Behind](#record-processing-falls-behind)
+ [Unauthorized KMS master key permission error](#unauthorized-kms-consumer)

## Some Kinesis Data Streams Records are Skipped When Using the Kinesis Client Library<a name="w7aac19c27b5"></a>

The most common cause of skipped records is an unhandled exception thrown from `processRecords`\. The Kinesis Client Library \(KCL\) relies on your `processRecords` code to handle any exceptions that arise from processing the data records\. Any exception thrown from `processRecords` is absorbed by the KCL\. To avoid infinite retries on a recurring failure, the KCL does not resend the batch of records processed at the time of the exception\. The KCL then calls `processRecords` for the next batch of data records without restarting the record processor\. This effectively results in consumer applications observing skipped records\. To prevent skipped records, handle all exceptions within `processRecords` appropriately\.

## Records Belonging to the Same Shard are Processed by Different Record Processors at the Same Time<a name="records-belonging-to-the-same-shard"></a>

For any running Kinesis Client Library \(KCL\) application, a shard only has one owner\. However, multiple record processors may temporarily process the same shard\. In the case of a worker instance that loses network connectivity, the KCL assumes that the unreachable worker is no longer processing records, after the failover time expires, and directs other worker instances to take over\. For a brief period, new record processors and record processors from the unreachable worker may process data from the same shard\. 

You should set a failover time that is appropriate for your application\. For low\-latency applications, the 10\-second default may represent the maximum time you want to wait\. However, in cases where you expect connectivity issues such as making calls across geographical areas where connectivity could be lost more frequently, this number may be too low\.

Your application should anticipate and handle this scenario, especially because network connectivity is usually restored to the previously unreachable worker\. If a record processor has its shards taken by another record processor, it must handle the following two cases to perform graceful shutdown:

1.  After the current call to `processRecords` is completed, the KCL invokes the shutdown method on the record processor with shutdown reason 'ZOMBIE'\. Your record processors are expected to clean up any resources as appropriate and then exit\.

1.  When you attempt to checkpoint from a 'zombie' worker, the KCL throws `ShutdownException`\. After receiving this exception, your code is expected to exit the current method cleanly\.

For more information, see [Handling Duplicate Records](kinesis-record-processor-duplicates.md)\.

## Consumer Application is Reading at a Slower Rate Than Expected<a name="consumer-app-reading-slower"></a>

The most common reasons for read throughput being slower than expected are as follows:

1. Multiple consumer applications have total reads exceeding the per\-shard limits\. For more information, see [Kinesis Data Streams Quotas and Limits](service-sizes-and-limits.md)\. In this case, increase the number of shards in the Kinesis data stream\.

1. The [limit](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html#API_GetRecords_RequestSyntax) that specifies the maximum number of GetRecords per call may have been configured with a low value\. If you are using the KCL, you may have configured the worker with a low value for the `maxRecords` property\. In general, we recommend using the system defaults for this property\.

1. The logic inside your `processRecords` call may be taking longer than expected for a number of possible reasons; the logic may be CPU intensive, I/O blocking, or bottlenecked on synchronization\. To test if this is true, test run empty record processors and compare the read throughput\. For information about how to keep up with the incoming data, see [Resharding, Scaling, and Parallel Processing](kinesis-record-processor-scaling.md)\.

If you have only one consumer application, it is always possible to read at least two times faster than the put rate\. That’s because you can write up to 1,000 records per second for writes, up to a maximum total data write rate of 1 MB per second \(including partition keys\)\. Each open shard can support up to 5 transactions per second for reads, up to a maximum total data read rate of 2 MB per second\. Note that each read \(GetRecords call\) gets a batch of records\. The size of the data returned by GetRecords varies depending on the utilization of the shard\. The maximum size of data that GetRecords can return is 10 MB\. If a call returns that limit, subsequent calls made within the next 5 seconds throw `ProvisionedThroughputExceededException`\.

## GetRecords Returns Empty Records Array Even When There is Data in the Stream<a name="getrecords-returns-empty"></a>

Consuming, or getting records is a pull model\. Developers are expected to call [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) in a continuous loop with no back\-offs\. Every call to GetRecords also returns a `ShardIterator` value, which must be used in the next iteration of the loop\. 

The GetRecords operation does not block\. Instead, it returns immediately; with either relevant data records or with an empty `Records` element\. An empty `Records` element is returned under two conditions: 

1. There is no more data currently in the shard\. 

1. There is no data near the part of the shard pointed to by the `ShardIterator`\.

The latter condition is subtle, but is a necessary design tradeoff to avoid unbounded seek time \(latency\) when retrieving records\. Thus, the stream\-consuming application should loop and call GetRecords, handling empty records as a matter of course\. 

In a production scenario, the only time the continuous loop should be exited is when the `NextShardIterator` value is `NULL`\. When `NextShardIterator` is `NULL`, it means that the current shard has been closed and the `ShardIterator`value would otherwise point past the last record\. If the consuming application never calls SplitShard or MergeShards, the shard remains open and the calls to GetRecords never return a `NextShardIterator` value that is `NULL`\. 

If you use the Kinesis Client Library \(KCL\), the above consumption pattern is abstracted for you\. This includes automatic handling of a set of shards that dynamically change\. With the KCL, the developer only supplies the logic to process incoming records\. This is possible because the library makes continuous calls to GetRecords for you\. 

## Shard Iterator Expires Unexpectedly<a name="shard-iterator-expires-unexpectedly"></a>

A new shard iterator is returned by every GetRecords request \(as `NextShardIterator`\), which you then use in the next GetRecords request \(as `ShardIterator`\)\. Typically, this shard iterator does not expire before you use it\. However, you may find that shard iterators expire because you have not called GetRecords for more than 5 minutes, or because you've performed a restart of your consumer application\.

If the shard iterator expires immediately, before you can use it, this might indicate that the DynamoDB table used by Kinesis does not have enough capacity to store the lease data\. This situation is more likely to happen if you have a large number of shards\. To solve this problem, increase the write capacity assigned to the shard table\. For more information, see [Using a Lease Table to Track the Shards Processed by the KCL Consumer Application](shared-throughput-kcl-consumers.md#shared-throughput-kcl-consumers-leasetable)\.

## Consumer Record Processing Falling Behind<a name="record-processing-falls-behind"></a>

For most use cases, consumer applications are reading the latest data from the stream\. In certain circumstances, consumer reads may fall behind, which may not be desired\. After you identify how far behind your consumers are reading, look at the most common reasons why consumers fall behind\. 

Start with the `GetRecords.IteratorAgeMilliseconds` metric, which tracks the read position across all shards and consumers in the stream\. Note that if an iterator's age passes 50% of the retention period \(by default 24 hours, configurable up to 7 days\), there is risk for data loss due to record expiration\. A quick stopgap solution is to increase the retention period\. This stops the loss of important data while you troubleshoot the issue further\. For more information, see [Monitoring the Amazon Kinesis Data Streams Service with Amazon CloudWatch](monitoring-with-cloudwatch.md)\. Next, identify how far behind your consumer application is reading from each shard using a custom CloudWatch metric emitted by the Kinesis Client Library \(KCL\), `MillisBehindLatest`\. For more information, see [Monitoring the Kinesis Client Library with Amazon CloudWatch](monitoring-with-kcl.md)\.

Here are the most common reasons consumers can fall behind:
+ Sudden large increases to `GetRecords.IteratorAgeMilliseconds` or `MillisBehindLatest` usually indicate a transient problem, such as API operation failures to a downstream application\. You should investigate these sudden increases if either of the metrics consistently display this behavior\. 
+ A gradual increase to these metrics indicates that a consumer is not keeping up with the stream because it is not processing records fast enough\. The most common root causes for this behavior are insufficient physical resources or record processing logic that has not scaled with an increase in stream throughput\. You can verify this behavior by looking at the other custom CloudWatch metrics that the KCL emits associated with the `processTask` operation, including `RecordProcessor.processRecords.Time`, `Success`, and `RecordsProcessed`\.
  + If you see an increase in the `processRecords.Time` metric that correlates with increased throughput, you should analyze your record processing logic to identify why it is not scaling with the increased throughput\.
  + If you see an increase to the `processRecords.Time` values that are not correlated with increased throughput, check to see if you are making any blocking calls in the critical path, which are often the cause of slowdowns in record processing\. An alternative approach is to increase your parallelism by increasing the number of shards\. Finally, confirm you have an adequate amount of physical resources \(memory, CPU utilization, etc\.\) on the underlying processing nodes during peak demand\.

## Unauthorized KMS master key permission error<a name="unauthorized-kms-consumer"></a>

This error occurs when a consumer application reads from an encrypted stream without permissions on the KMS master key\. To assign permissions to an application to access a KMS key, see [Using Key Policies in AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) and [Using IAM Policies with AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html)\.