# Monitoring the Amazon Kinesis Data Streams Service with Amazon CloudWatch<a name="monitoring-with-cloudwatch"></a>

Amazon Kinesis Data Streams and Amazon CloudWatch are integrated so that you can collect, view, and analyze CloudWatch metrics for your Kinesis data streams\. For example, to keep track of shard usage, you can monitor the `PutRecords.Bytes` and `GetRecords.Bytes` metrics and compare them to the number of shards in the stream\.

The metrics that you configure for your streams are automatically collected and pushed to CloudWatch every minute\. Metrics are archived for two weeks; after that period, the data is discarded\.

The following table describes basic stream\-level and enhanced shard\-level monitoring for Kinesis streams\.


| Type | Description | 
| --- | --- | 
|  Basic \(stream\-level\)  |  Stream\-level data is sent automatically every minute at no charge\.  | 
|  Enhanced \(shard\-level\)  |  Shard\-level data is sent every minute for an additional cost\. To get this level of data, you must specifically enable it for the stream using the [EnableEnhancedMonitoring](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_EnableEnhancedMonitoring.html) operation\.  For information about pricing, see the [Amazon CloudWatch product page](https://aws.amazon.com/cloudwatch)\.  | 

## Amazon Kinesis Data Streams Dimensions and Metrics<a name="kinesis-metrics"></a>

Kinesis Data Streams sends metrics to CloudWatch at two levels; the stream level and, optionally, the shard level\. Stream\-level metrics are for most common monitoring use cases in normal conditions\. Shard\-level metrics are for specific monitoring tasks, usually related to troubleshooting, and are enabled using the [EnableEnhancedMonitoring](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_EnableEnhancedMonitoring.html) operation\. 

For an explanation of the statistics gathered from CloudWatch metrics, see [CloudWatch Statistics](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/cloudwatch_concepts.html#Statistic) in the *Amazon CloudWatch User Guide*\.

**Topics**
+ [Basic Stream\-level Metrics](#kinesis-metrics-stream)
+ [Enhanced Shard\-level Metrics](#kinesis-metrics-shard)
+ [Dimensions for Amazon Kinesis Data Streams Metrics](#kinesis-metricdimensions)
+ [Recommended Amazon Kinesis Data Streams Metrics](#kinesis-metric-use)

### Basic Stream\-level Metrics<a name="kinesis-metrics-stream"></a>

The `AWS/Kinesis` namespace includes the following stream\-level metrics\.

Kinesis Data Streams sends these stream\-level metrics to CloudWatch every minute\. These metrics are always available\.


| Metric | Description | 
| --- | --- | 
| GetRecords\.Bytes |  The number of bytes retrieved from the Kinesis stream, measured over the specified time period\. Minimum, Maximum, and Average statistics represent the bytes in a single `GetRecords` operation for the stream in the specified time period\. Shard\-level metric name: `OutgoingBytes` Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Bytes  | 
| GetRecords\.IteratorAge |  This metric is deprecated\. Use `GetRecords.IteratorAgeMilliseconds`\.  | 
| GetRecords\.IteratorAgeMilliseconds |  The age of the last record in all `GetRecords` calls made against an Kinesis stream, measured over the specified time period\. Age is the difference between the current time and when the last record of the `GetRecords` call was written to the stream\. The Minimum and Maximum statistics can be used to track the progress of Kinesis consumer applications\. A value of zero indicates that the records being read are completely caught up with the stream\. Shard\-level metric name: `IteratorAgeMilliseconds` Dimensions: StreamName Statistics: Minimum, Maximum, Average, Samples Units: Milliseconds  | 
| GetRecords\.Latency |  The time taken per `GetRecords` operation, measured over the specified time period\. Dimensions: StreamName Statistics: Minimum, Maximum, Average Units: Milliseconds  | 
| GetRecords\.Records |  The number of records retrieved from the shard, measured over the specified time period\. Minimum, Maximum, and Average statistics represent the records in a single `GetRecords` operation for the stream in the specified time period\. Shard\-level metric name: `OutgoingRecords` Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 
| GetRecords\.Success |  The number of successful `GetRecords` operations per stream, measured over the specified time period\. Dimensions: StreamName Statistics: Average, Sum, Samples Units: Count  | 
| IncomingBytes |  The number of bytes successfully put to the Kinesis stream over the specified time period\. This metric includes bytes from `PutRecord` and `PutRecords` operations\. Minimum, Maximum, and Average statistics represent the bytes in a single put operation for the stream in the specified time period\. Shard\-level metric name: `IncomingBytes` Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Bytes  | 
| IncomingRecords |  The number of records successfully put to the Kinesis stream over the specified time period\. This metric includes record counts from `PutRecord` and `PutRecords` operations\. Minimum, Maximum, and Average statistics represent the records in a single put operation for the stream in the specified time period\. Shard\-level metric name: `IncomingRecords` Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 
| PutRecord\.Bytes |  The number of bytes put to the Kinesis stream using the `PutRecord` operation over the specified time period\. Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Bytes  | 
| PutRecord\.Latency |  The time taken per `PutRecord` operation, measured over the specified time period\. Dimensions: StreamName Statistics: Minimum, Maximum, Average Units: Milliseconds  | 
| PutRecord\.Success |  The number of successful `PutRecord` operations per Kinesis stream, measured over the specified time period\. Average reflects the percentage of successful writes to a stream\. Dimensions: StreamName Statistics: Average, Sum, Samples Units: Count  | 
| PutRecords\.Bytes |  The number of bytes put to the Kinesis stream using the `PutRecords` operation over the specified time period\. Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Bytes  | 
| PutRecords\.Latency |  The time taken per `PutRecords` operation, measured over the specified time period\. Dimensions: StreamName Statistics: Minimum, Maximum, Average Units: Milliseconds  | 
| PutRecords\.Records |  The number of successful records in a `PutRecords` operation per Kinesis stream, measured over the specified time period\. Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 
| PutRecords\.Success |  The number of `PutRecords` operations where at least one record succeeded, per Kinesis stream, measured over the specified time period\. Dimensions: StreamName Statistics: Average, Sum, Samples Units: Count  | 
| ReadProvisionedThroughputExceeded |  The number of `GetRecords` calls throttled for the stream over the specified time period\. The most commonly used statistic for this metric is Average\. When the Minimum statistic has a value of 1, all records were throttled for the stream during the specified time period\.  When the Maximum statistic has a value of 0 \(zero\), no records were throttled for the stream during the specified time period\. Shard\-level metric name: `ReadProvisionedThroughputExceeded` Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 
| WriteProvisionedThroughputExceeded |  The number of records rejected due to throttling for the stream over the specified time period\. This metric includes throttling from `PutRecord` and `PutRecords` operations\. The most commonly used statistic for this metric is Average\. When the Minimum statistic has a non\-zero value, records were being throttled for the stream during the specified time period\.  When the Maximum statistic has a value of 0 \(zero\), no records were being throttled for the stream during the specified time period\. Shard\-level metric name: `WriteProvisionedThroughputExceeded` Dimensions: StreamName Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 

### Enhanced Shard\-level Metrics<a name="kinesis-metrics-shard"></a>

The `AWS/Kinesis` namespace includes the following shard\-level metrics\.

Kinesis sends the following shard\-level metrics to CloudWatch every minute\. These metrics are not enabled by default\. There is a charge for enhanced metrics emitted from Kinesis\. For more information, see [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/) under the heading *Amazon CloudWatch Custom Metrics*\. The charges are given per shard per metric per month\.


| Metric | Description | 
| --- | --- | 
| IncomingBytes |  The number of bytes successfully put to the shard over the specified time period\. This metric includes bytes from `PutRecord` and `PutRecords` operations\. Minimum, Maximum, and Average statistics represent the bytes in a single put operation for the shard in the specified time period\. Stream\-level metric name: `IncomingBytes` Dimensions: StreamName, ShardId Statistics: Minimum, Maximum, Average, Sum, Samples Units: Bytes  | 
| IncomingRecords |  The number of records successfully put to the shard over the specified time period\. This metric includes record counts from `PutRecord` and `PutRecords` operations\. Minimum, Maximum, and Average statistics represent the records in a single put operation for the shard in the specified time period\. Stream\-level metric name: `IncomingRecords` Dimensions: StreamName, ShardId Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 
| IteratorAgeMilliseconds |  The age of the last record in all `GetRecords` calls made against a shard, measured over the specified time period\. Age is the difference between the current time and when the last record of the `GetRecords` call was written to the stream\. The Minimum and Maximum statistics can be used to track the progress of Kinesis consumer applications\. A value of 0 \(zero\) indicates that the records being read are completely caught up with the stream\. Stream\-level metric name: `GetRecords.IteratorAgeMilliseconds` Dimensions: StreamName, ShardId Statistics: Minimum, Maximum, Average, Samples Units: Milliseconds  | 
| OutgoingBytes |  The number of bytes retrieved from the shard, measured over the specified time period\. Minimum, Maximum, and Average statistics represent the bytes in a single `GetRecords` operation for the shard in the specified time period\. Stream\-level metric name: `GetRecords.Bytes` Dimensions: StreamName, ShardId Statistics: Minimum, Maximum, Average, Sum, Samples Units: Bytes  | 
| OutgoingRecords |  The number of records retrieved from the shard, measured over the specified time period\. Minimum, Maximum, and Average statistics represent the records in a single `GetRecords` operation for the shard in the specified time period\. Stream\-level metric name: `GetRecords.Records` Dimensions: StreamName, ShardId Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 
| ReadProvisionedThroughputExceeded |  The number of `GetRecords` calls throttled for the shard over the specified time period\. This exception count covers all dimensions of the following limits: 5 reads per shard per second or 2 MB per second per shard\. The most commonly used statistic for this metric is Average\. When the Minimum statistic has a value of 1, all records were throttled for the shard during the specified time period\.  When the Maximum statistic has a value of 0 \(zero\), no records were throttled for the shard during the specified time period\. Stream\-level metric name: `ReadProvisionedThroughputExceeded` Dimensions: StreamName, ShardId Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 
| WriteProvisionedThroughputExceeded |  The number of records rejected due to throttling for the shard over the specified time period\. This metric includes throttling from `PutRecord` and `PutRecords` operations and covers all dimensions of the following limits: 1,000 records per second per shard or 1 MB per second per shard\. The most commonly used statistic for this metric is Average\. When the Minimum statistic has a non\-zero value, records were being throttled for the shard during the specified time period\.  When the Maximum statistic has a value of 0 \(zero\), no records were being throttled for the shard during the specified time period\. Stream\-level metric name: `WriteProvisionedThroughputExceeded` Dimensions: StreamName, ShardId Statistics: Minimum, Maximum, Average, Sum, Samples Units: Count  | 

### Dimensions for Amazon Kinesis Data Streams Metrics<a name="kinesis-metricdimensions"></a>

You can use the following dimensions to filter the metrics for Amazon Kinesis Data Streams\.


| Dimension | Description | 
| --- | --- | 
|  `StreamName`  |  The name of the Kinesis stream\.  | 
|  `ShardId`  |  The shard ID within the Kinesis stream\.  | 

### Recommended Amazon Kinesis Data Streams Metrics<a name="kinesis-metric-use"></a>

There are several Amazon Kinesis Data Streams metrics of particular interest to the majority of Kinesis Data Streams customers\. The following list provides recommended metrics and their uses\.


| Metric | Usage Notes | 
| --- | --- | 
|  `GetRecords.IteratorAgeMilliseconds`  |  Tracks the read position across all shards and consumers in the stream\. Note that if an iterator's age passes 50% of the retention period \(by default 24 hours, configurable up to 7 days\), there is risk for data loss due to record expiration\. We advise the use of CloudWatch alarms on the Maximum statistic to alert you before this loss is a risk\. For an example scenario that uses this metric, see [Consumer Record Processing Falling Behind](troubleshooting-consumers.md#record-processing-falls-behind)\.  | 
|  `ReadProvisionedThroughputExceeded`  |  When your consumer side record processing is falling behind, it is sometimes difficult to know where the bottleneck is\. Use this metric to determine if your reads are being throttled due to exceeding your read throughput limits\. The most commonly used statistic for this metric is Average\.  | 
| WriteProvisionedThroughputExceeded | This is for the same purpose as the ReadProvisionedThroughputExceeded metric, but for the producer \(put\) side of the stream\. The most commonly used statistic for this metric is Average\. | 
| PutRecord\.Success, PutRecords\.Success | We advise the use of CloudWatch alarms on the Average statistic to indicate if records are failing to the stream\. Choose one or both put types depending on what your producer uses\. If using the Kinesis Producer Library \(KPL\), use PutRecords\.Success\. | 
| GetRecords\.Success | We advise the use of CloudWatch alarms on the Average statistic to indicate if records are failing from the stream\. | 

## Accessing Amazon CloudWatch Metrics for Kinesis Data Streams<a name="cloudwatch-metrics"></a>

You can monitor metrics for Kinesis Data Streams using the CloudWatch console, the command line, or the CloudWatch API\. The following procedures show you how to access metrics using these different methods\. 

**To access metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. From the navigation bar, select a region\.

1. In the navigation pane, choose **Metrics**\.

1. In the **CloudWatch Metrics by Category** pane, choose **Kinesis Metrics**\.

1. Click the relevant row to view the statistics for the specified **MetricName** and **StreamName**\. 

   **Note:** Most console statistic names match the corresponding CloudWatch metric names listed above, with the exception of **Read Throughput** and **Write Throughput**\. These statistics are calculated over 5\-minute intervals: **Write Throughput** monitors the `IncomingBytes` CloudWatch metric, and **Read Throughput** monitors `GetRecords.Bytes`\.

1. \(Optional\) In the graph pane, select a statistic and a time period and then create a CloudWatch alarm using these settings\.

**To access metrics using the AWS CLI**  
Use the [list\-metrics](http://docs.aws.amazon.com/cli/latest/reference/cloudwatch/list-metrics.html) and [get\-metric\-statistics](http://docs.aws.amazon.com/cli/latest/reference/cloudwatch/get-metric-statistics.html) commands\.

**To access metrics using the CloudWatch CLI**  
Use the [mon\-list\-metrics](http://docs.aws.amazon.com/AmazonCloudWatch/latest/cli/cli-mon-list-metrics.html) and [mon\-get\-stats](http://docs.aws.amazon.com/AmazonCloudWatch/latest/cli/cli-mon-get-stats.html) commands\.

**To access metrics using the CloudWatch API**  
Use the [ListMetrics](http://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_ListMetrics.html) and [GetMetricStatistics](http://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricStatistics.html) operations\.