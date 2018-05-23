# Developing Amazon Kinesis Data Streams Consumers Using the Kinesis Client Library<a name="developing-consumers-with-kcl"></a>

You can develop a consumer application for Amazon Kinesis Data Streams using the Kinesis Client Library \(KCL\)\. Although you can use the Kinesis Data Streams API to get data from a Kinesis data stream, we recommend using the design patterns and code for consumer applications provided by the KCL\.

You can monitor the KCL with Amazon CloudWatch\. For more information, see [Monitoring the Kinesis Client Library with Amazon CloudWatch](monitoring-with-kcl.md)\.

**Topics**
+ [Kinesis Client Library](#kinesis-record-processor-overview-kcl)
+ [Role of the KCL](#kinesis-record-processor-kcl-role)
+ [Developing a Kinesis Client Library Consumer in Java](kinesis-record-processor-implementation-app-java.md)
+ [Developing a Kinesis Client Library Consumer in Node\.js](kinesis-record-processor-implementation-app-nodejs.md)
+ [Developing a Kinesis Client Library Consumer in \.NET](kinesis-record-processor-implementation-app-dotnet.md)
+ [Developing a Kinesis Client Library Consumer in Python](kinesis-record-processor-implementation-app-py.md)
+ [Developing a Kinesis Client Library Consumer in Ruby](kinesis-record-processor-implementation-app-ruby.md)

## Kinesis Client Library<a name="kinesis-record-processor-overview-kcl"></a>

The Kinesis Client Library \(KCL\) helps you consume and process data from a Kinesis data stream\. This type of application is also referred to as a *consumer*\. The KCL takes care of many of the complex tasks associated with distributed computing, such as load\-balancing across multiple instances, responding to instance failures, checkpointing processed records, and reacting to resharding\. The KCL enables you to focus on writing record processing logic\. 

Note that the KCL is different from the Kinesis Data Streams API that is available in the AWS SDKs\. The Kinesis Data Streams API helps you manage many aspects of Kinesis Data Streams \(including creating streams, resharding, and putting and getting records\), while the KCL provides a layer of abstraction specifically for processing data in a consumer role\. For information about the Kinesis Data Streams API, see the [Amazon Kinesis API Reference](http://docs.aws.amazon.com/kinesis/latest/APIReference/)\.

The KCL is a Java library; support for languages other than Java is provided using a multi\-language interface called the *MultiLangDaemon*\. This daemon is Java\-based and runs in the background when you are using a KCL language other than Java\. For example, if you install the KCL for Python and write your consumer app entirely in Python, you still need Java installed on your system because of the MultiLangDaemon\. Further, MultiLangDaemon has some default settings you may need to customize for your use case, for example the AWS region it connects to\. For more information about the MultiLangDaemon, go to the [KCL MultiLangDaemon project](https://github.com/awslabs/amazon-kinesis-client/tree/master/src/main/java/com/amazonaws/services/kinesis/multilang) page on GitHub\.

At run time, a KCL application instantiates a worker with configuration information, and then uses a record processor to process the data received from a Kinesis data stream\. You can run a KCL application on any number of instances\. Multiple instances of the same application coordinate on failures and load\-balance dynamically\. You can also have multiple KCL applications working on the same stream, subject to throughput limits\.

## Role of the KCL<a name="kinesis-record-processor-kcl-role"></a>

The KCL acts as an intermediary between your record processing logic and Kinesis Data Streams\.

When you start a KCL application, it calls the KCL to instantiate a *worker*\. This call provides the KCL with configuration information for the application, such as the stream name and AWS credentials\.

The KCL performs the following tasks:
+ Connects to the stream 
+ Enumerates the shards 
+ Coordinates shard associations with other workers \(if any\) 
+ Instantiates a record processor for every shard it manages 
+ Pulls data records from the stream 
+ Pushes the records to the corresponding record processor 
+ Checkpoints processed records 
+ Balances shard\-worker associations when the worker instance count changes
+ Balances shard\-worker associations when shards are split or merged