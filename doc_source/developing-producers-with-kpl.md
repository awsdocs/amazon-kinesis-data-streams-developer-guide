# Developing Producers Using the Amazon Kinesis Producer Library<a name="developing-producers-with-kpl"></a>

An Amazon Kinesis Data Streams producer is an application that puts user data records into a Kinesis data stream \(also called *data ingestion*\)\. The Kinesis Producer Library \(KPL\) simplifies producer application development, allowing developers to achieve high write throughput to a Kinesis data stream\. 

You can monitor the KPL with Amazon CloudWatch\. For more information, see [Monitoring the Kinesis Producer Library with Amazon CloudWatch](monitoring-with-kpl.md)\.

**Topics**
+ [Role of the KPL](#developing-producers-with-kpl-role)
+ [Advantages of Using the KPL](#developing-producers-with-kpl-advantage)
+ [When Not to Use the KPL](#developing-producers-with-kpl-when)
+ [Installing the KPL](kinesis-kpl-dl-install.md)
+ [Transitioning to Amazon Trust Services \(ATS\) Certificates for the Kinesis Producer Library](kinesis-kpl-upgrades.md)
+ [KPL Supported Platforms](kinesis-kpl-supported-plats.md)
+ [KPL Key Concepts](kinesis-kpl-concepts.md)
+ [Integrating the KPL with Producer Code](kinesis-kpl-integration.md)
+ [Writing to your Kinesis Data Stream Using the KPL](kinesis-kpl-writing.md)
+ [Configuring the Kinesis Producer Library](kinesis-kpl-config.md)
+ [Consumer De\-aggregation](kinesis-kpl-consumer-deaggregation.md)
+ [Using the KPL with Kinesis Data Firehose](kpl-with-firehose.md)

## Role of the KPL<a name="developing-producers-with-kpl-role"></a>

The KPL is an easy\-to\-use, highly configurable library that helps you write to a Kinesis data stream\. It acts as an intermediary between your producer application code and the Kinesis Data Streams API actions\. The KPL performs the following primary tasks: 
+ Writes to one or more Kinesis data streams with an automatic and configurable retry mechanism
+ Collects records and uses `PutRecords` to write multiple records to multiple shards per request
+ Aggregates user records to increase payload size and improve throughput
+ Integrates seamlessly with the [Kinesis Client Library](https://docs.aws.amazon.com/kinesis/latest/dev/developing-consumers-with-kcl.html) \(KCL\) to de\-aggregate batched records on the consumer
+ Submits Amazon CloudWatch metrics on your behalf to provide visibility into producer performance

Note that the KPL is different from the Kinesis Data Streams API that is available in the [AWS SDKs](https://aws.amazon.com/tools/)\. The Kinesis Data Streams API helps you manage many aspects of Kinesis Data Streams \(including creating streams, resharding, and putting and getting records\), while the KPL provides a layer of abstraction specifically for ingesting data\. For information about the Kinesis Data Streams API, see the [Amazon Kinesis API Reference](https://docs.aws.amazon.com/kinesis/latest/APIReference/)\.

## Advantages of Using the KPL<a name="developing-producers-with-kpl-advantage"></a>

The following list represents some of the major advantages to using the KPL for developing Kinesis Data Streams producers\.

The KPL can be used in either synchronous or asynchronous use cases\. We suggest using the higher performance of the asynchronous interface unless there is a specific reason to use synchronous behavior\. For more information about these two use cases and example code, see [Writing to your Kinesis Data Stream Using the KPL](kinesis-kpl-writing.md)\.

**Performance Benefits**  
The KPL can help build high\-performance producers\. Consider a situation where your Amazon EC2 instances serve as a proxy for collecting 100\-byte events from hundreds or thousands of low power devices and writing records into a Kinesis data stream\. These EC2 instances must each write thousands of events per second to your data stream\. To achieve the throughput needed, producers must implement complicated logic, such as batching or multithreading, in addition to retry logic and record de\-aggregation at the consumer side\. The KPL performs all of these tasks for you\. 

**Consumer\-Side Ease of Use**  
For consumer\-side developers using the KCL in Java, the KPL integrates without additional effort\. When the KCL retrieves an aggregated Kinesis Data Streams record consisting of multiple KPL user records, it automatically invokes the KPL to extract the individual user records before returning them to the user\.   
For consumer\-side developers who do not use the KCL but instead use the API operation `GetRecords` directly, a KPL Java library is available to extract the individual user records before returning them to the user\. 

**Producer Monitoring**   
You can collect, monitor, and analyze your Kinesis Data Streams producers using Amazon CloudWatch and the KPL\. The KPL emits throughput, error, and other metrics to CloudWatch on your behalf, and is configurable to monitor at the stream, shard, or producer level\.

**Asynchronous Architecture**   
Because the KPL may buffer records before sending them to Kinesis Data Streams, it does not force the caller application to block and wait for a confirmation that the record has arrived at the server before continuing execution\. A call to put a record into the KPL always returns immediately and does not wait for the record to be sent or a response to be received from the server\. Instead, a `Future` object is created that receives the result of sending the record to Kinesis Data Streams at a later time\. This is the same behavior as asynchronous clients in the AWS SDK\.

## When Not to Use the KPL<a name="developing-producers-with-kpl-when"></a>

The KPL can incur an additional processing delay of up to `RecordMaxBufferedTime` within the library \(user\-configurable\)\. Larger values of `RecordMaxBufferedTime` results in higher packing efficiencies and better performance\. Applications that cannot tolerate this additional delay may need to use the AWS SDK directly\. For more information about using the AWS SDK with Kinesis Data Streams, see [Developing Producers Using the Amazon Kinesis Data Streams API with the AWS SDK for Java](developing-producers-with-sdk.md)\. For more information about `RecordMaxBufferedTime` and other user\-configurable properties of the KPL, see [Configuring the Kinesis Producer Library](kinesis-kpl-config.md)\.