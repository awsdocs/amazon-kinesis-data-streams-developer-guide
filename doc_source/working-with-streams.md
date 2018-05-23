# Managing Kinesis Data Streams Using Java<a name="working-with-streams"></a>

These examples discuss the [Amazon Kinesis Data Streams API](http://docs.aws.amazon.com/kinesis/latest/APIReference/) and use the [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) to create, delete, and work with a Kinesis data stream\.

The Java example code in this chapter demonstrates how to perform basic Kinesis Data Streams API operations, and are divided up logically by operation type\. These examples do not represent production\-ready code, in that they do not check for all possible exceptions, or account for all possible security or performance considerations\. Also, you can call the [Kinesis Data Streams API](http://docs.aws.amazon.com/kinesis/latest/APIReference/) using other different programming languages\. For more information about all available AWS SDKs, see [Start Developing with Amazon Web Services](https://aws.amazon.com/developers/getting-started/)\.

**Topics**
+ [Creating a Stream](kinesis-using-sdk-java-create-stream.md)
+ [Listing Streams](kinesis-using-sdk-java-list-streams.md)
+ [Retrieving Shards from a Stream](kinesis-using-sdk-java-retrieve-shards.md)
+ [Deleting a Stream](kinesis-using-sdk-java-delete-stream.md)
+ [Resharding a Stream](kinesis-using-sdk-java-resharding.md)
+ [Changing the Data Retention Period](kinesis-extended-retention.md)