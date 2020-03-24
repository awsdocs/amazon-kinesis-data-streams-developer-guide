# Managing Enhanced Fan\-Out Consumers with the AWS Management Console<a name="building-enhanced-consumers-console"></a>

Consumers that use *enhanced fan\-out* in Amazon Kinesis Data Streams can receive records from a data stream with dedicated throughput of up to 2 MB of data per second per shard\. For more information, see [Developing Custom Consumers with Dedicated Throughput \(Enhanced Fan\-Out\)](enhanced-consumers.md)\.

You can use the AWS Management Console to see a list of all the consumers that are registered to use enhanced fan\-out with a specific stream\. For each such consumer, you can see details such as ARN, status, and creation date, in addition to the monitoring metrics and the tags associated with the consumer\.

**To view consumers that are registered to use enhanced fan\-out, their status, creation date, and metrics on the console**

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation pane\.

1. Choose a Kinesis data stream to view its details\.

1. On the details page for the stream, choose the **Enhanced fan\-out** tab\.

1. Choose a consumer to see its name, status, and date of registration\.

**To deregister a consumer**

1. Open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation pane\.

1. Choose a Kinesis data stream to view its details\.

1. On the details page for the stream, choose the **Enhanced fan\-out** tab\.

1. Select the check box to the left of the name of every consumer that you want to deregister\.

1. Choose **Deregister consumer**\.