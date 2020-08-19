# Developing a Kinesis Client Library Consumer in Python<a name="kinesis-record-processor-implementation-app-py"></a>

You can use the Kinesis Client Library \(KCL\) to build applications that process data from your Kinesis data streams\. The Kinesis Client Library is available in multiple languages\. This topic discusses Python\.

The KCL is a Java library; support for languages other than Java is provided using a multi\-language interface called the *MultiLangDaemon*\. This daemon is Java\-based and runs in the background when you are using a KCL language other than Java\. Therefore, if you install the KCL for Python and write your consumer app entirely in Python, you still need Java installed on your system because of the MultiLangDaemon\. Further, MultiLangDaemon has some default settings you may need to customize for your use case, for example, the AWS Region that it connects to\. For more information about the MultiLangDaemon on GitHub, go to the [KCL MultiLangDaemon project](https://github.com/awslabs/amazon-kinesis-client/tree/v1.x/src/main/java/com/amazonaws/services/kinesis/multilang) page\.

To download the Python KCL from GitHub, go to [Kinesis Client Library \(Python\)](https://github.com/awslabs/amazon-kinesis-client-python)\. To download sample code for a Python KCL consumer application, go to the [KCL for Python sample project](https://github.com/awslabs/amazon-kinesis-client-python/tree/master/samples) page on GitHub\.

You must complete the following tasks when implementing a KCL consumer application in Python:

**Topics**
+ [Implement the RecordProcessor Class Methods](#kinesis-record-processor-implementation-interface-py)
+ [Modify the Configuration Properties](#kinesis-record-processor-initialization-py)

## Implement the RecordProcessor Class Methods<a name="kinesis-record-processor-implementation-interface-py"></a>

The `RecordProcess` class must extend the `RecordProcessorBase` to implement the following methods\. The sample provides implementations that you can use as a starting point \(see `sample_kclpy_app.py`\)\.

```
def initialize(self, shard_id)
def process_records(self, records, checkpointer)
def shutdown(self, checkpointer, reason)
```

**initialize**  
 The KCL calls the `initialize` method when the record processor is instantiated, passing a specific shard ID as a parameter\. This record processor processes only this shard, and typically, the reverse is also true \(this shard is processed only by this record processor\)\. However, your consumer should account for the possibility that a data record might be processed more than one time\. This is because Kinesis Data Streams has *at least once* semantics, meaning that every data record from a shard is processed at least one time by a worker in your consumer\. For more information about cases in which a particular shard may be processed by more than one worker, see [Resharding, Scaling, and Parallel Processing](kinesis-record-processor-scaling.md)\.

```
def initialize(self, shard_id)
```

**process\_records**  
 The KCL calls this method, passing a list of data record from the shard specified by the `initialize` method\. The record processor that you implement processes the data in these records according to the semantics of your consumer\. For example, the worker might perform a transformation on the data and then store the result in an Amazon Simple Storage Service \(Amazon S3\) bucket\.

```
def process_records(self, records, checkpointer) 
```

In addition to the data itself, the record also contains a sequence number and partition key\. The worker can use these values when processing the data\. For example, the worker could choose the S3 bucket in which to store the data based on the value of the partition key\. The `record` dictionary exposes the following key\-value pairs to access the record's data, sequence number, and partition key:

```
record.get('data')
record.get('sequenceNumber')
record.get('partitionKey')
```

Note that the data is Base64\-encoded\.

In the sample, the method `process_records` has code that shows how a worker can access the record's data, sequence number, and partition key\.

Kinesis Data Streams requires the record processor to keep track of the records that have already been processed in a shard\. The KCL takes care of this tracking for you by passing a `Checkpointer` object to `process_records`\. The record processor calls the `checkpoint` method on this object to inform the KCL of how far it has progressed in processing the records in the shard\. If the worker fails, the KCL uses this information to restart the processing of the shard at the last known processed record\.

For a split or merge operation, the KCL doesn't start processing the new shards until the processors for the original shards have called `checkpoint` to signal that all processing on the original shards is complete\.

If you don't pass a parameter, the KCL assumes that the call to `checkpoint` means that all records have been processed, up to the last record that was passed to the record processor\. Therefore, the record processor should call `checkpoint` only after it has processed all the records in the list that was passed to it\. Record processors do not need to call `checkpoint` on each call to `process_records`\. A processor could, for example, call `checkpoint` on every third call\. You can optionally specify the exact sequence number of a record as a parameter to `checkpoint`\. In this case, the KCL assumes that all records have been processed up to that record only\.

In the sample, the private method `checkpoint` shows how to call the `Checkpointer.checkpoint` method using appropriate exception handling and retry logic\.

The KCL relies on `process_records` to handle any exceptions that arise from processing the data records\. If an exception is thrown from `process_records`, the KCL skips over the data records that were passed to `process_records` before the exception\. That is, these records are not re\-sent to the record processor that threw the exception or to any other record processor in the consumer\.

**shutdown**  
 The KCL calls the `shutdown` method either when processing ends \(the shutdown reason is `TERMINATE`\) or the worker is no longer responding \(the shutdown `reason` is `ZOMBIE`\)\.

```
def shutdown(self, checkpointer, reason)
```

Processing ends when the record processor does not receive any further records from the shard, because either the shard was split or merged, or the stream was deleted\.

 The KCL also passes a `Checkpointer` object to `shutdown`\. If the shutdown `reason` is `TERMINATE`, the record processor should finish processing any data records, and then call the `checkpoint` method on this interface\.

## Modify the Configuration Properties<a name="kinesis-record-processor-initialization-py"></a>

The sample provides default values for the configuration properties\. You can override any of these properties with your own values \(see `sample.properties`\)\.

### Application Name<a name="kinesis-record-processor-application-name-py"></a>

The KCL requires an application name that is unique among your applications, and among Amazon DynamoDB tables in the same Region\. It uses the application name configuration value in the following ways:
+ All workers that are associated with this application name are assumed to be working together on the same stream\. These workers can be distributed on multiple instances\. If you run an additional instance of the same application code, but with a different application name, the KCL treats the second instance as an entirely separate application that is also operating on the same stream\.
+ The KCL creates a DynamoDB table with the application name and uses the table to maintain state information \(such as checkpoints and worker\-shard mapping\) for the application\. Each application has its own DynamoDB table\. For more information, see [Using a Lease Table to Track the Shards Processed by the KCL Consumer Application](shared-throughput-kcl-consumers.md#shared-throughput-kcl-consumers-leasetable)\.

### Set Up Credentials<a name="kinesis-record-processor-creds-py"></a>

You must make your AWS credentials available to one of the credential providers in the default credential providers chain\. You can you use the `AWSCredentialsProvider` property to set a credentials provider\. The [sample\.properties](https://github.com/awslabs/amazon-kinesis-client-python/blob/master/samples/sample.properties) must make your credentials available to one of the credentials providers in the [default credential providers chain](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)\. If you are running your consumer application on an Amazon EC2 instance, we recommend that you configure the instance with an IAM role\. AWS credentials that reflect the permissions associated with this IAM role are made available to applications on the instance through its instance metadata\. This is the most secure way to manage credentials for a consumer application running on an EC2 instance\.

The sample's properties file configures KCL to process a Kinesis data stream called "words" using the record processor supplied in `sample_kclpy_app.py`\. 