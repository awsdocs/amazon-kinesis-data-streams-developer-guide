# What Is Amazon Kinesis Data Streams?<a name="introduction"></a>

You can use Amazon Kinesis Data Streams to collect and process large [streams](https://aws.amazon.com/streaming-data/) of data records in real time\. You can create data\-processing applications, known as *Kinesis Data Streams applications*\. A typical Kinesis Data Streams application reads data from a *data stream* as data records\. These applications can use the Kinesis Client Library, and they can run on Amazon EC2 instances\. You can send the processed records to dashboards, use them to generate alerts, dynamically change pricing and advertising strategies, or send data to a variety of other AWS services\. For information about Kinesis Data Streams features and pricing, see [Amazon Kinesis Data Streams](https://aws.amazon.com/kinesis/streams/)\.

Kinesis Data Streams is part of the Kinesis streaming data platform, along with [Kinesis Data Firehose](https://docs.aws.amazon.com/firehose/latest/dev/), [Kinesis Video Streams](https://docs.aws.amazon.com/kinesisvideostreams/latest/dg/), and [Kinesis Data Analytics](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/)\.

For more information about AWS big data solutions, see [Big Data on AWS](https://aws.amazon.com/big-data/)\. For more information about AWS streaming data solutions, see [What is Streaming Data?](https://aws.amazon.com/streaming-data/)\.

**Topics**
+ [What Can I Do with Kinesis Data Streams?](#use-service-for-what)
+ [Benefits of Using Kinesis Data Streams](#using-the-service)
+ [Related Services](#related-services)
+ [Creating and Updating Data Streams](amazon-kinesis-streams.md)
+ [Kinesis Data Streams Producers](amazon-kinesis-producers.md)
+ [Kinesis Data Streams Consumers](amazon-kinesis-consumers.md)
+ [Kinesis Data Streams Quotas and Limits](service-sizes-and-limits.md)

## What Can I Do with Kinesis Data Streams?<a name="use-service-for-what"></a>

You can use Kinesis Data Streams for rapid and continuous data intake and aggregation\. The type of data used can include IT infrastructure log data, application logs, social media, market data feeds, and web clickstream data\. Because the response time for the data intake and processing is in real time, the processing is typically lightweight\.

The following are typical scenarios for using Kinesis Data Streams:

Accelerated log and data feed intake and processing  
You can have producers push data directly into a stream\. For example, push system and application logs and they are available for processing in seconds\. This prevents the log data from being lost if the front end or application server fails\. Kinesis Data Streams provides accelerated data feed intake because you don't batch the data on the servers before you submit it for intake\.

Real\-time metrics and reporting  
You can use data collected into Kinesis Data Streams for simple data analysis and reporting in real time\. For example, your data\-processing application can work on metrics and reporting for system and application logs as the data is streaming in, rather than wait to receive batches of data\.

Real\-time data analytics  
This combines the power of parallel processing with the value of real\-time data\. For example, process website clickstreams in real time, and then analyze site usability engagement using multiple different Kinesis Data Streams applications running in parallel\.

Complex stream processing  
You can create Directed Acyclic Graphs \(DAGs\) of Kinesis Data Streams applications and data streams\. This typically involves putting data from multiple Kinesis Data Streams applications into another stream for downstream processing by a different Kinesis Data Streams application\.

## Benefits of Using Kinesis Data Streams<a name="using-the-service"></a>

Although you can use Kinesis Data Streams to solve a variety of streaming data problems, a common use is the real\-time aggregation of data followed by loading the aggregate data into a data warehouse or map\-reduce cluster\.

Data is put into Kinesis data streams, which ensures durability and elasticity\. The delay between the time a record is put into the stream and the time it can be retrieved \(put\-to\-get delay\) is typically less than 1 second\. In other words, a Kinesis Data Streams application can start consuming the data from the stream almost immediately after the data is added\. The managed service aspect of Kinesis Data Streams relieves you of the operational burden of creating and running a data intake pipeline\. You can create streaming map\-reduce–type applications\. The elasticity of Kinesis Data Streams enables you to scale the stream up or down, so that you never lose data records before they expire\.

Multiple Kinesis Data Streams applications can consume data from a stream, so that multiple actions, like archiving and processing, can take place concurrently and independently\. For example, two applications can read data from the same stream\. The first application calculates running aggregates and updates an Amazon DynamoDB table, and the second application compresses and archives data to a data store like Amazon Simple Storage Service \(Amazon S3\)\. The DynamoDB table with running aggregates is then read by a dashboard for up\-to\-the\-minute reports\.

The Kinesis Client Library enables fault\-tolerant consumption of data from streams and provides scaling support for Kinesis Data Streams applications\.

## Related Services<a name="related-services"></a>

For information about using Amazon EMR clusters to read and process Kinesis data streams directly, see [Kinesis Connector](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-kinesis.html)\.