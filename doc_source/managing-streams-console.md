# Managing Kinesis Data Streams Using the Console<a name="managing-streams-console"></a>

The following procedures show you how to create, delete, and work with an Amazon Kinesis data stream using the AWS Management Console\.

**To create a stream**

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation bar\.

1. Choose **Create Kinesis stream**\.

1. Enter a name for the stream \(for example, **StockTradeStream**\)\.

1. Specify the number of shards\. If you need help, expand **Estimate the number of shards you'll need**\.

1. Choose **Create Kinesis stream**\.

**To list your streams**

1. Open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation bar\.

1. \(Optional\) To view more details for a stream, choose the name of the stream\.

**To edit a stream**

1. Open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation bar\.

1. Choose the name of the stream\.

1. To scale the shard capacity, do the following:

   1. Under **Shards**, choose **Edit**\.

   1. Specify the new number of shards\.

   1. Choose **Save**\.

1. To edit the data retention period, do the following:

   1. Under **Data retention period**, choose **Edit**\.

   1. Specify a period between 24 and 168 hours\. Records are stored in the stream for this period of time\. Additional charges apply for periods greater than 24 hours\. For more information, see [Amazon Kinesis Data Streams pricing](https://aws.amazon.com/kinesis/streams/pricing/)\.

   1. Choose **Save**\.

1. To enable or disable shard\-level metrics, do the following:

   1. Under **Shard level metrics**, choose **Edit**\.

   1. Select the metrics to monitor\. For more information, see [Enhanced Shard\-level Metrics](monitoring-with-cloudwatch.md#kinesis-metrics-shard)\.

   1. Choose **Save**\.

**To delete your streams**

1. Open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation bar\.

1. Select the check box next to the streams to delete\.

1. Choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Delete**\.