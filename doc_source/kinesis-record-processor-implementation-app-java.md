# Developing a Kinesis Client Library Consumer in Java<a name="kinesis-record-processor-implementation-app-java"></a>

You can use the Kinesis Client Library \(KCL\) to build applications that process data from your Kinesis data streams\. The Kinesis Client Library is available in multiple languages\. This topic discusses Java\. To view the Javadoc reference, see the [AWS Javadoc topic for Class AmazonKinesisClient](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kinesis/AmazonKinesisClient.html)\.

To download the Java KCL from GitHub, go to [Kinesis Client Library \(Java\)](https://github.com/awslabs/amazon-kinesis-client)\. To locate the Java KCL on Apache Maven, go to the [KCL search results](https://search.maven.org/#search|ga|1|amazon-kinesis-client) page\. To download sample code for a Java KCL consumer application from GitHub, go to the [KCL for Java sample project](https://github.com/aws/aws-sdk-java/tree/master/src/samples/AmazonKinesis) page on GitHub\. 

The sample application uses [Apache Commons Logging](http://commons.apache.org/proper/commons-logging/guide.html)\. You can change the logging configuration in the static `configure` method defined in the `AmazonKinesisApplicationSample.java` file\. For more information about how to use Apache Commons Logging with Log4j and AWS Java applications, see [Logging with Log4j](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-logging.html) in the *AWS SDK for Java Developer Guide*\.

You must complete the following tasks when implementing a KCL consumer application in Java:

**Topics**
+ [Implement the IRecordProcessor Methods](#kinesis-record-processor-implementation-interface-java)
+ [Implement a Class Factory for the IRecordProcessor Interface](#kinesis-record-processor-implementation-factory-java)
+ [Create a Worker](#kcl-java-worker)
+ [Modify the Configuration Properties](#kinesis-record-processor-initialization-java)
+ [Migrating to Version 2 of the Record Processor Interface](#kcl-java-v2-migration)

## Implement the IRecordProcessor Methods<a name="kinesis-record-processor-implementation-interface-java"></a>

The KCL currently supports two versions of the `IRecordProcessor` interface:The original interface is available with the first version of the KCL, and version 2 is available starting with KCL version 1\.5\.0\. Both interfaces are fully supported\. Your choice depends on your specific scenario requirements\. Refer to your locally built Javadocs or the source code to see all the differences\. The following sections outline the minimal implementation for getting started\.

**Topics**
+ [Original Interface \(Version 1\)](#kcl-java-interface-original)
+ [Updated Interface \(Version 2\)](#kcl-java-interface-v2)

### Original Interface \(Version 1\)<a name="kcl-java-interface-original"></a>

The original `IRecordProcessor` interface \(`package com.amazonaws.services.kinesis.clientlibrary.interfaces`\) exposes the following record processor methods that your consumer must implement\. The sample provides implementations that you can use as a starting point \(see `AmazonKinesisApplicationSampleRecordProcessor.java`\)\.

```
public void initialize(String shardId)
public void processRecords(List<Record> records, IRecordProcessorCheckpointer checkpointer)
public void shutdown(IRecordProcessorCheckpointer checkpointer, ShutdownReason reason)
```

**initialize**  
The KCL calls the `initialize` method when the record processor is instantiated, passing a specific shard ID as a parameter\. This record processor processes only this shard and typically, the reverse is also true \(this shard is processed only by this record processor\)\. However, your consumer should account for the possibility that a data record might be processed more than one time\. Kinesis Data Streams has *at least once* semantics, meaning that every data record from a shard is processed at least one time by a worker in your consumer\. For more information about cases in which a particular shard may be processed by more than one worker, see [Resharding, Scaling, and Parallel Processing](kinesis-record-processor-scaling.md)\.

```
public void initialize(String shardId)
```

**processRecords**  
The KCL calls the `processRecords` method, passing a list of data record from the shard specified by the `initialize(shardId)` method\. The record processor processes the data in these records according to the semantics of the consumer\. For example, the worker might perform a transformation on the data and then store the result in an Amazon Simple Storage Service \(Amazon S3\) bucket\.

```
public void processRecords(List<Record> records, IRecordProcessorCheckpointer checkpointer) 
```

In addition to the data itself, the record also contains a sequence number and partition key\. The worker can use these values when processing the data\. For example, the worker could choose the S3 bucket in which to store the data based on the value of the partition key\. The `Record` class exposes the following methods that provide access to the record's data, sequence number, and partition key\. 

```
record.getData()  
record.getSequenceNumber() 
record.getPartitionKey()
```

In the sample, the private method `processRecordsWithRetries` has code that shows how a worker can access the record's data, sequence number, and partition key\.

Kinesis Data Streams requires the record processor to keep track of the records that have already been processed in a shard\. The KCL takes care of this tracking for you by passing a checkpointer \(`IRecordProcessorCheckpointer`\) to `processRecords`\. The record processor calls the `checkpoint` method on this interface to inform the KCL of how far it has progressed in processing the records in the shard\. If the worker fails, the KCL uses this information to restart the processing of the shard at the last known processed record\.

For a split or merge operation, the KCL won't start processing the new shards until the processors for the original shards have called `checkpoint` to signal that all processing on the original shards is complete\.

If you don't pass a parameter, the KCL assumes that the call to `checkpoint` means that all records have been processed, up to the last record that was passed to the record processor\. Therefore, the record processor should call `checkpoint` only after it has processed all the records in the list that was passed to it\. Record processors do not need to call `checkpoint` on each call to `processRecords`\. A processor could, for example, call `checkpoint` on every third call to `processRecords`\. You can optionally specify the exact sequence number of a record as a parameter to `checkpoint`\. In this case, the KCL assumes that all records have been processed up to that record only\.

In the sample, the private method `checkpoint` shows how to call `IRecordProcessorCheckpointer.checkpoint` using the appropriate exception handling and retry logic\.

The KCL relies on `processRecords` to handle any exceptions that arise from processing the data records\. If an exception is thrown from `processRecords`, the KCL skips over the data records that were passed before the exception\. That is, these records are not re\-sent to the record processor that threw the exception or to any other record processor in the consumer\.

**shutdown**  
The KCL calls the `shutdown` method either when processing ends \(the shutdown reason is `TERMINATE`\) or the worker is no longer responding \(the shutdown reason is `ZOMBIE`\)\.

```
public void shutdown(IRecordProcessorCheckpointer checkpointer, ShutdownReason reason)
```

Processing ends when the record processor does not receive any further records from the shard, because either the shard was split or merged, or the stream was deleted\.

The KCL also passes a `IRecordProcessorCheckpointer` interface to `shutdown`\. If the shutdown reason is `TERMINATE`, the record processor should finish processing any data records, and then call the `checkpoint` method on this interface\.

### Updated Interface \(Version 2\)<a name="kcl-java-interface-v2"></a>

The updated `IRecordProcessor` interface \(`package com.amazonaws.services.kinesis.clientlibrary.interfaces.v2`\) exposes the following record processor methods that your consumer must implement: 

```
void initialize(InitializationInput initializationInput)
void processRecords(ProcessRecordsInput processRecordsInput)
void shutdown(ShutdownInput shutdownInput)
```

All of the arguments from the original version of the interface are accessible through get methods on the container objects\. For example, to retrieve the list of records in `processRecords()`, you can use `processRecordsInput.getRecords()`\.

As of version 2 of this interface \(KCL 1\.5\.0 and later\), the following new inputs are available in addition to the inputs provided by the original interface:

starting sequence number  
In the `InitializationInput` object passed to the `initialize()` operation, the starting sequence number from which records would be provided to the record processor instance\. This is the sequence number that was last checkpointed by the record processor instance previously processing the same shard\. This is provided in case your application needs this information\. 

pending checkpoint sequence number  
In the `InitializationInput` object passed to the `initialize()` operation, the pending checkpoint sequence number \(if any\) that could not be committed before the previous record processor instance stopped\.

## Implement a Class Factory for the IRecordProcessor Interface<a name="kinesis-record-processor-implementation-factory-java"></a>

You also need to implement a factory for the class that implements the record processor methods\. When your consumer instantiates the worker, it passes a reference to this factory\.

The sample implements the factory class in the file `AmazonKinesisApplicationSampleRecordProcessorFactory.java` using the original record processor interface\. If you want the class factory to create version 2 record processors, use the package name `com.amazonaws.services.kinesis.clientlibrary.interfaces.v2`\.

```
  public class SampleRecordProcessorFactory implements IRecordProcessorFactory { 
      /**
      * Constructor.
      */
      public SampleRecordProcessorFactory() {
          super();
      }
      /**
      * {@inheritDoc}
      */
      @Override
      public IRecordProcessor createProcessor() {
          return new SampleRecordProcessor();
      }
  }
```

## Create a Worker<a name="kcl-java-worker"></a>

As discussed in [Implement the IRecordProcessor Methods](#kinesis-record-processor-implementation-interface-java), there are two versions of the KCL record processor interface to choose from, which affects how you would create a worker\. The original record processor interface uses the following code structure to create a worker:

```
final KinesisClientLibConfiguration config = new KinesisClientLibConfiguration(...)
final IRecordProcessorFactory recordProcessorFactory = new RecordProcessorFactory();
final Worker worker = new Worker(recordProcessorFactory, config);
```

With version 2 of the record processor interface, you can use `Worker.Builder` to create a worker without needing to worry about which constructor to use and the order of the arguments\. The updated record processor interface uses the following code structure to create a worker:

```
final KinesisClientLibConfiguration config = new KinesisClientLibConfiguration(...)
final IRecordProcessorFactory recordProcessorFactory = new RecordProcessorFactory();
final Worker worker = new Worker.Builder()
    .recordProcessorFactory(recordProcessorFactory)
    .config(config)
    .build();
```

## Modify the Configuration Properties<a name="kinesis-record-processor-initialization-java"></a>

The sample provides default values for configuration properties\. This configuration data for the worker is then consolidated in a `KinesisClientLibConfiguration` object\. This object and a reference to the class factory for `IRecordProcessor` are passed in the call that instantiates the worker\. You can override any of these properties with your own values using a Java properties file \(see `AmazonKinesisApplicationSample.java`\)\.

### Application Name<a name="configuration-property-application-name"></a>

The KCL requires an application name that is unique across your applications, and across Amazon DynamoDB tables in the same Region\. It uses the application name configuration value in the following ways:
+ All workers associated with this application name are assumed to be working together on the same stream\. These workers may be distributed on multiple instances\. If you run an additional instance of the same application code, but with a different application name, the KCL treats the second instance as an entirely separate application that is also operating on the same stream\.
+ The KCL creates a DynamoDB table with the application name and uses the table to maintain state information \(such as checkpoints and worker\-shard mapping\) for the application\. Each application has its own DynamoDB table\. For more information, see [Using a Lease Table to Track the Shards Processed by the KCL Consumer Application](shared-throughput-kcl-consumers.md#shared-throughput-kcl-consumers-leasetable)\.

### Set Up Credentials<a name="kinesis-record-processor-cred-java"></a>

You must make your AWS credentials available to one of the credential providers in the default credential providers chain\. For example, if you are running your consumer on an EC2 instance, we recommend that you launch the instance with an IAM role\. AWS credentials that reflect the permissions associated with this IAM role are made available to applications on the instance through its instance metadata\. This is the most secure way to manage credentials for a consumer running on an EC2 instance\.

The sample application first attempts to retrieve IAM credentials from instance metadata: 

```
credentialsProvider = new InstanceProfileCredentialsProvider(); 
```

If the sample application cannot obtain credentials from the instance metadata, it attempts to retrieve credentials from a properties file:

```
credentialsProvider = new ClasspathPropertiesFileCredentialsProvider();
```

For more information about instance metadata, see [Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) in the *Amazon EC2 User Guide for Linux Instances*\.

### Use Worker ID for Multiple Instances<a name="kinesis-record-processor-workerid-java"></a>

The sample initialization code creates an ID for the worker, `workerId`, using the name of the local computer and appending a globally unique identifier as shown in the following code snippet\. This approach supports the scenario of multiple instances of the consumer application running on a single computer\.

```
String workerId = InetAddress.getLocalHost().getCanonicalHostName() + ":" + UUID.randomUUID();
```

## Migrating to Version 2 of the Record Processor Interface<a name="kcl-java-v2-migration"></a>

If you want to migrate code that uses the original interface, in addition to the steps described previously, the following steps are required:

1. Change your record processor class to import the version 2 record processor interface:

   ```
   import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessor;
   ```

1. Change the references to inputs to use `get` methods on the container objects\. For example, in the `shutdown()` operation, change "`checkpointer`" to "`shutdownInput.getCheckpointer()`"\.

1. Change your record processor factory class to import the version 2 record processor factory interface:

   ```
   import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessorFactory;
   ```

1. Change the construction of the worker to use `Worker.Builder`\. For example:

   ```
   final Worker worker = new Worker.Builder()
       .recordProcessorFactory(recordProcessorFactory)
       .config(config)
       .build();
   ```