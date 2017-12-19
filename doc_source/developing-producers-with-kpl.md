# Developing Amazon Kinesis Streams Producers Using the Kinesis Producer Library<a name="developing-producers-with-kpl"></a>

An Amazon Kinesis Streams producer is any application that puts user data records into an Kinesis stream \(also called *data ingestion*\)\. The Kinesis Producer Library \(KPL\) simplifies producer application development, allowing developers to achieve high write throughput to a Kinesis stream\. 

You can monitor the KPL with Amazon CloudWatch\. For more information, see [Monitoring the Kinesis Producer Library with Amazon CloudWatch](monitoring-with-kpl.md)\.


+ [Role of the KPL](#w3ab1c11b7b7b9)
+ [Advantages of Using the KPL](#w3ab1c11b7b7c11)
+ [When Not To Use the KPL](#w3ab1c11b7b7c13)
+ [Installation](kinesis-kpl-dl-install.md)
+ [Supported Platforms](kinesis-kpl-supported-plats.md)
+ [Key Concepts](kinesis-kpl-concepts.md)
+ [Integration With Producer Code](kinesis-kpl-integration.md)
+ [Writing to your Kinesis Streams Stream Using the KPL](kinesis-kpl-writing.md)
+ [Configuring the KPL](kinesis-kpl-config.md)
+ [Consumer De\-aggregation](kinesis-kpl-consumer-deaggregation.md)

## Role of the KPL<a name="w3ab1c11b7b7b9"></a>

The KPL is an easy\-to\-use, highly configurable library that helps you write to a Kinesis stream\. It acts as an intermediary between your producer application code and the Kinesis Streams API actions\. The KPL performs the following primary tasks: 

+ Writes to one or more Kinesis streams with an automatic and configurable retry mechanism

+ Collects records and uses `PutRecords` to write multiple records to multiple shards per request

+ Aggregates user records to increase payload size and improve throughput

+ Integrates seamlessly with the [Kinesis Client Library](http://docs.aws.amazon.com/kinesis/latest/dev/developing-consumers-with-kcl.html) \(KCL\) to de\-aggregate batched records on the consumer

+ Submits Amazon CloudWatch metrics on your behalf to provide visibility into producer performance

Note that the KPL is different from the Kinesis Streams API that is available in the [AWS SDKs](https://aws.amazon.com/tools/)\. The Kinesis Streams API helps you manage many aspects of Kinesis Streams \(including creating streams, resharding, and putting and getting records\), while the KPL provides a layer of abstraction specifically for ingesting data\. For information about the Kinesis Streams API, see the [Amazon Kinesis API Reference](http://docs.aws.amazon.com/kinesis/latest/APIReference/)\.

## Advantages of Using the KPL<a name="w3ab1c11b7b7c11"></a>

The following list represents some of the major advantages to using the KPL for developing Kinesis Streams producers\.

The KPL can be used in either synchronous or asynchronous use cases\. We suggest using the higher performance of the asynchronous interface unless there is a specific reason to use synchronous behavior\. For more information about these two use cases and example code, see [Writing to your Kinesis Streams Stream Using the KPL](kinesis-kpl-writing.md)\.

**Performance Benefits**  
The KPL can help build high\-performance producers\. Consider a situation where your Amazon EC2 instances serve as a proxy for collecting 100\-byte events from hundreds or thousands of low power devices and writing records into an Kinesis stream\. These EC2 instances must each write thousands of events per second to your Kinesis stream\. To achieve the throughput needed, producers must implement complicated logic such as batching or multithreading, in addition to retry logic and record de\-aggregation at the consumer side\. The KPL performs all of these tasks for you\. 

**Consumer\-side Ease of Use**  
For consumer\-side developers using the KCL in Java, the KPL integrates without additional effort\. When the KCL retrieves an aggregated Kinesis Streams record consisting of multiple KPL user records, it automatically invokes the KPL to extract the individual user records before returning them to the user\.   
For consumer\-side developers who do not use the KCL but instead use the API operation `GetRecords` directly, a KPL Java library is available to extract the individual user records before returning them to the user\. 

**Producer Monitoring**   
You can collect, monitor, and analyze your Kinesis Streams producers using Amazon CloudWatch and the KPL\. The KPL emits throughput, error, and other metrics to CloudWatch on your behalf, and is configurable to monitor at the stream, shard, or producer level\.

**Asynchronous Architecture**   
Because the KPL may buffer records before sending them to Kinesis Streams, it does not force the caller application to block and wait for a confirmation that the record has arrived at the server before continuing execution\. A call to put a record into the KPL always returns immediately and does not wait for the record to be sent or a response to be received from the server\. Instead, a `Future` object is created that receives the result of sending the record to Kinesis Streams at a later time\. This is the same behavior as asynchronous clients in the AWS SDK\.

## When Not To Use the KPL<a name="w3ab1c11b7b7c13"></a>

The KPL can incur an additional processing delay of up to `RecordMaxBufferedTime` within the library \(user\-configurable\)\. Larger values of `RecordMaxBufferedTime` results in higher packing efficiencies and better performance\. Applications that cannot tolerate this additional delay may need to use the AWS SDK directly\. For more information about using the AWS SDK with Kinesis Streams, see [Developing Amazon Kinesis Streams Producers Using the Amazon Kinesis Streams API with the AWS SDK for Java](developing-producers-with-sdk.md)\. For more information about `RecordMaxBufferedTime` and other user\-configurable properties of the KPL, see [Configuring the KPL](kinesis-kpl-config.md)\.