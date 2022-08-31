# Updating a Stream<a name="updating-a-stream"></a>

You can update the details of a stream using the Kinesis Data Streams console, the Kinesis Data Streams API, or the AWS CLI\.

**Note**  
You can enable server\-side encryption for existing streams, or for streams that you have recently created\.

## <a name="update-stream-console"></a>

**To update a data stream using the console**

1. Open the Amazon Kinesis console at [https://console\.aws\.amazon\.com/kinesis/](https://console.aws.amazon.com/kinesis/)\.

1. In the navigation bar, expand the Region selector and choose a Region\.

1. Choose the name of your stream in the list\. The **Stream Details** page displays a summary of your stream configuration and monitoring information\.

1. To switch between on\-demand and provisioned capacity modes for a data stream, choose **Edit capacity mode** in the **Configuration** tab\. For more information, see [Choosing the Data Stream Capacity Mode](how-do-i-size-a-stream.md)\.
**Important**  
For each data stream in your AWS account, you can switch between the on\-demand and provisioned modes twice within 24 hours\.

1. For a data stream with the provisioned mode, to edit the number of shards, choose **Edit provisioned shards** in the **Configuration** tab, and then enter a new shard count\.

1. To enable server\-side encryption of data records, choose **Edit** in the **Server\-side encryption** section\. Choose a KMS key to use as the master key for encryption, or use the default master key, **aws/kinesis**, managed by Kinesis\. If you enable encryption for a stream and use your own AWS KMS master key, ensure that your producer and consumer applications have access to the AWS KMS master key that you used\. To assign permissions to an application to access a user\-generated AWS KMS key, see [Permissions to Use User\-Generated KMS Master Keys](permissions-user-key-KMS.md)\.

1. To edit the data retention period, choose **Edit** in the **Data retention period** section, and then enter a new data retention period\.

1. If you have enabled custom metrics on your account, choose **Edit** in the **Shard level metrics** section, and then specify metrics for your stream\. For more information, see [Monitoring the Amazon Kinesis Data Streams Service with Amazon CloudWatch](monitoring-with-cloudwatch.md)\.

## Updating a Stream Using the API<a name="update-stream-api"></a>

To update stream details using the API, see the following methods:
+ [AddTagsToStream](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_AddTagsToStream.html)
+ [DecreaseStreamRetentionPeriod](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_DecreaseStreamRetentionPeriod.html)
+ [DisableEnhancedMonitoring](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_DisableEnhancedMonitoring.html)
+ [EnableEnhancedMonitoring](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_EnableEnhancedMonitoring.html)
+ [IncreaseStreamRetentionPeriod](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_IncreaseStreamRetentionPeriod.html)
+ [RemoveTagsFromStream](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_RemoveTagsFromStream.html)
+ [StartStreamEncryption](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_StartStreamEncryption.html)
+ [StopStreamEncryption](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_StopStreamEncryption.html)
+ [UpdateShardCount](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html)

## Updating a Stream Using the AWS CLI<a name="update-stream-cli"></a>

For information about updating a stream using the AWS CLI, see the [Kinesis CLI reference](https://docs.aws.amazon.com/cli/latest/reference/kinesis/index.html)\. 