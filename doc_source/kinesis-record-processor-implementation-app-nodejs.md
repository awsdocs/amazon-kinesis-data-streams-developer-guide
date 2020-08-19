# Developing a Kinesis Client Library Consumer in Node\.js<a name="kinesis-record-processor-implementation-app-nodejs"></a>

You can use the Kinesis Client Library \(KCL\) to build applications that process data from your Kinesis data streams\. The Kinesis Client Library is available in multiple languages\. This topic discusses Node\.js\.

The KCL is a Java library; support for languages other than Java is provided using a multi\-language interface called the *MultiLangDaemon*\. This daemon is Java\-based and runs in the background when you are using a KCL language other than Java\. Therefore, if you install the KCL for Node\.js and write your consumer app entirely in Node\.js, you still need Java installed on your system because of the MultiLangDaemon\. Further, MultiLangDaemon has some default settings you may need to customize for your use case, for example, the AWS Region that it connects to\. For more information about the MultiLangDaemon on GitHub, go to the [KCL MultiLangDaemon project](https://github.com/awslabs/amazon-kinesis-client/tree/v1.x/src/main/java/com/amazonaws/services/kinesis/multilang) page\.

To download the Node\.js KCL from GitHub, go to [Kinesis Client Library \(Node\.js\)](https://github.com/awslabs/amazon-kinesis-client-nodejs)\.

**Sample Code Downloads**

There are two code samples available for KCL in Node\.js:
+ [basic\-sample](https://github.com/awslabs/amazon-kinesis-client-nodejs/tree/master/samples/basic_sample)

  Used in the following sections to illustrate the fundamentals of building a KCL consumer application in Node\.js\.
+ [click\-stream\-sample](https://github.com/awslabs/amazon-kinesis-client-nodejs/tree/master/samples/click_stream_sample)

   Slightly more advanced and uses a real\-world scenario, after you have familiarized yourself with the basic sample code\. This sample is not discussed here but has a README file with more information\.

You must complete the following tasks when implementing a KCL consumer application in Node\.js:

**Topics**
+ [Implement the Record Processor](#kinesis-record-processor-implementation-interface-nodejs)
+ [Modify the Configuration Properties](#kinesis-record-processor-initialization-nodejs)

## Implement the Record Processor<a name="kinesis-record-processor-implementation-interface-nodejs"></a>

The simplest possible consumer using the KCL for Node\.js must implement a `recordProcessor` function, which in turn contains the functions `initialize`, `processRecords`, and `shutdown`\. The sample provides an implementation that you can use as a starting point \(see `sample_kcl_app.js`\)\.

```
function recordProcessor() {
  // return an object that implements initialize, processRecords and shutdown functions.}
```

**initialize**  
The KCL calls the `initialize` function when the record processor starts\. This record processor processes only the shard ID passed as `initializeInput.shardId`, and typically, the reverse is also true \(this shard is processed only by this record processor\)\. However, your consumer should account for the possibility that a data record might be processed more than one time\. This is because Kinesis Data Streams has *at least once* semantics, meaning that every data record from a shard is processed at least one time by a worker in your consumer\. For more information about cases in which a particular shard might be processed by more than one worker, see [Resharding, Scaling, and Parallel Processing](kinesis-record-processor-scaling.md)\.

```
initialize: function(initializeInput, completeCallback)
```

**processRecords**  
 The KCL calls this function with input that contains a list of data records from the shard specified to the `initialize` function\. The record processor that you implement processes the data in these records according to the semantics of your consumer\. For example, the worker might perform a transformation on the data and then store the result in an Amazon Simple Storage Service \(Amazon S3\) bucket\. 

```
processRecords: function(processRecordsInput, completeCallback)
```

In addition to the data itself, the record also contains a sequence number and partition key, which the worker can use when processing the data\. For example, the worker could choose the S3 bucket in which to store the data based on the value of the partition key\. The `record` dictionary exposes the following key\-value pairs to access the record's data, sequence number, and partition key:

```
record.data
record.sequenceNumber
record.partitionKey
```

Note that the data is Base64\-encoded\.

In the basic sample, the function `processRecords` has code that shows how a worker can access the record's data, sequence number, and partition key\.

Kinesis Data Streams requires the record processor to keep track of the records that have already been processed in a shard\. The KCL takes care of this tracking for with a `checkpointer` object passed as `processRecordsInput.checkpointer`\. Your record processor calls the `checkpointer.checkpoint` function to inform the KCL how far it has progressed in processing the records in the shard\. In the event that the worker fails, the KCL uses this information when you restart the processing of the shard so that it continues from the last known processed record\.

For a split or merge operation, the KCL doesn't start processing the new shards until the processors for the original shards have called `checkpoint` to signal that all processing on the original shards is complete\.

If you don't pass the sequence number to the `checkpoint` function, the KCL assumes that the call to `checkpoint` means that all records have been processed, up to the last record that was passed to the record processor\. Therefore, the record processor should call `checkpoint` **only** after it has processed all the records in the list that was passed to it\. Record processors do not need to call `checkpoint` on each call to `processRecords`\. A processor could, for example, call `checkpoint` on every third call, or some event external to your record processor, such as a custom verification/validation service you've implemented\. 

You can optionally specify the exact sequence number of a record as a parameter to `checkpoint`\. In this case, the KCL assumes that all records have been processed up to that record only\.

The basic sample application shows the simplest possible call to the `checkpointer.checkpoint` function\. You can add other checkpointing logic you need for your consumer at this point in the function\.

**shutdown**  
The KCL calls the `shutdown` function either when processing ends \(`shutdownInput.reason` is `TERMINATE`\) or the worker is no longer responding \(`shutdownInput.reason` is `ZOMBIE`\)\.

```
shutdown: function(shutdownInput, completeCallback)
```

Processing ends when the record processor does not receive any further records from the shard, because either the shard was split or merged, or the stream was deleted\.

The KCL also passes a `shutdownInput.checkpointer` object to `shutdown`\. If the shutdown reason is `TERMINATE`, you should make sure that the record processor has finished processing any data records, and then call the `checkpoint` function on this interface\.

## Modify the Configuration Properties<a name="kinesis-record-processor-initialization-nodejs"></a>

The sample provides default values for the configuration properties\. You can override any of these properties with your own values \(see `sample.properties` in the basic sample\)\.

### Application Name<a name="kinesis-record-processor-application-name-nodejs"></a>

The KCL requires an application that this is unique among your applications, and among Amazon DynamoDB tables in the same Region\. It uses the application name configuration value in the following ways:
+ All workers associated with this application name are assumed to be working together on the same stream\. These workers may be distributed on multiple instances\. If you run an additional instance of the same application code, but with a different application name, the KCL treats the second instance as an entirely separate application that is also operating on the same stream\.
+ The KCL creates a DynamoDB table with the application name and uses the table to maintain state information \(such as checkpoints and worker\-shard mapping\) for the application\. Each application has its own DynamoDB table\. For more information, see [Using a Lease Table to Track the Shards Processed by the KCL Consumer Application](shared-throughput-kcl-consumers.md#shared-throughput-kcl-consumers-leasetable)\.

### Set Up Credentials<a name="kinesis-record-processor-credentials-nodejs"></a>

You must make your AWS credentials available to one of the credential providers in the default credential providers chain\. You can you use the `AWSCredentialsProvider` property to set a credentials provider\. The `sample.properties` file must make your credentials available to one of the credentials providers in the [default credential providers chain](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)\. If you are running your consumer on an Amazon EC2 instance, we recommend that you configure the instance with an IAM role\. AWS credentials that reflect the permissions associated with this IAM role are made available to applications on the instance through its instance metadata\. This is the most secure way to manage credentials for a consumer application running on an EC2 instance\.

The following example configures KCL to process a Kinesis data stream named `kclnodejssample` using the record processor supplied in `sample_kcl_app.js`:

```
# The Node.js executable script
executableName = node sample_kcl_app.js
# The name of an Amazon Kinesis stream to process
streamName = kclnodejssample
# Unique KCL application name
applicationName = kclnodejssample
# Use default AWS credentials provider chain
AWSCredentialsProvider = DefaultAWSCredentialsProviderChain
# Read from the beginning of the stream
initialPositionInStream = TRIM_HORIZON
```