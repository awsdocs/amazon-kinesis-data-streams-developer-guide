# Choosing the Data Stream Capacity Mode<a name="how-do-i-size-a-stream"></a>

**Topics**
+ [What is a Data Stream Capacity Mode?](#whatiscapacitymode)
+ [On\-demand Mode](#ondemandmode)
+ [Provisioned Mode](#provisionedmode)
+ [Switching Between Capacity Modes](#switchingmodes)

## What is a Data Stream Capacity Mode?<a name="whatiscapacitymode"></a>

A capacity mode determines how the capacity of a data stream is managed and how you are charged for the usage of your data stream\. In Amazon Kinesis Data Streams, you can choose between an **on\-demand** mode and a **provisioned** mode for your data streams\. 
+ **On\-demand** \- data streams with an on\-demand mode require no capacity planning and automatically scale to handle gigabytes of write and read throughput per minute\. With the on\-demand mode, Kinesis Data Streams automatically manages the shards in order to provide the necessary throughput\. 
+ **Provisioned** \- for the data streams with a provisioned mode, you must specify the number of shards for the data stream\. The total capacity of a data stream is the sum of the capacities of its shards\. You can increase or decrease the number of shards in a data stream as needed\.

You can use Kinesis Data Streams `PutRecord` and `PutRecords` APIs to write data into your data streams in both on\-demand and provisioned capacity modes\. To retrieve data, both capacity modes support default consumers that use the `GetRecords` API and Enhanced Fan\-Out \(EFO\) consumers that use the `SubscribeToShard` API\.

All Kinesis Data Streams capabilities, including retention mode, encryption, monitoring metrics, and others, are supported for both the on\-demand and provisioned modes\. Kinesis Data Streams provides the high durability and availability in both the on\-demand and provisioned capacity modes\. 

## On\-demand Mode<a name="ondemandmode"></a>

Data streams in the on\-demand mode require no capacity planning and automatically scale to handle gigabytes of write and read throughput per minute\. On\-demand mode simplifies ingesting and storing large data volumes at a low\-latency because it eliminates provisioning and managing servers, storage, or throughput\. You can ingest billions of records per day without any operational overhead\.

On\-demand mode is ideal for addressing the needs of highly variable and unpredictable application traffic\. You no longer have to provision these workloads for peak capacity, which can result in higher costs due to low utilization\. On\-demand mode is suited for workloads with unpredictable and highly\-variable traffic patterns\. 

With the on\-demand capacity mode, you pay per GB of data written and read from your data streams\. You do not need to specify how much read and write throughput you expect your application to perform\. Kinesis Data Streams instantly accommodates your workloads as they ramp up or down\. For more information, see [Amazon Kinesis Data Streams pricing](https://aws.amazon.com/kinesis/data-streams/pricing/)\.

You can create a new data stream with the on\-demand mode by using the Kinesis Data Streams console, APIs, or CLI commands\. 

A data stream in the on\-demand mode accommodates up to double the peak write throughput observed in the previous 30 days\. As your data stream’s write throughput reaches a new peak, Kinesis Data Streams scales the data stream’s capacity automatically\. For example, if your data stream has a write throughput that varies between 10 MB/s and 40 MB/s, then Kinesis Data Streams ensures that you can easily burst to double your previous peak throughput, or 80 MB/s\. If the same data stream sustains a new peak throughput of 50 MB/s, Kinesis Data Streams ensures that there is enough capacity to ingest 100 MB/s of write throughput\. However, write throttling can occur if your traffic increases to more than double the previous peak within a 15\-minute duration\. You need to retry these throttled requests\.

The aggregate read capacity of a data stream with the on\-demand mode increases proportionally to write throughput\. This helps to ensure that consumer applications always have adequate read throughput to process incoming data in real time\. You get at least twice the write throughput compared to read data using the `GetRecords` API\. We recommend that you use one consumer application with the `GetRecord` API, so that it has enough room to catch up when the application needs to recover from downtime\. It is recommended that you use the Enhanced Fan\-Out capability of Kinesis Data Streams for scenarios that require adding more than one consumer application\. Enhanced Fan\-Out supports adding up to 20 consumer applications to a data stream using the `SubscribeToShard` API, with each consumer application having dedicated throughput\. 

### Handling Read and Write Throughput Exceptions<a name="hotshards"></a>

With the on\-demand capacity mode \(same as with the provisioned capacity mode\), you must specify a partition key with each record to write data into your data stream\. Kinesis Data Streams uses your partition keys to distribute data across shards\. Kinesis Data Streams monitors traffic for each shard\. When the incoming traffic exceeds 500 KB/s per shard, it splits the shard within 15 minutes\. The parent shard’s hash key values are redistributed evenly across child shards\.

 If your incoming traffic exceeds twice your prior peak, you can experience read or write exceptions for about 15 minutes, even when your data is distributed evenly across the shards\. We recommend that you retry all such requests so that all the records are properly stored in Kinesis Data Streams\. 

You may experience read and write exceptions if you are using a partition key that leads to uneven data distribution, and the records assigned to a particular shard exceed its limits\. With on\-demand mode, the data stream automatically adapts to handle uneven data distribution patterns unless a single partition key exceeds a shard’s 1 MB/s throughput and 1000 records per second limits\. 

In the on\-demand mode, Kinesis Data Streams splits the shards evenly when it detects an increase in traffic\. However, it does not detect and isolate hash keys that are driving a higher portion of incoming traffic to a particular shard\. If you are using highly uneven partition keys you may continue to receive write exceptions\. For such use cases, we recommend that you use the provisioned capacity mode that supports granular shard splits\.

## Provisioned Mode<a name="provisionedmode"></a>

With provisioned mode, after you create the data stream, you can dynamically scale your shard capacity up or down using the AWS Management Console or the [UpdateShardCount](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html) API\. You can make updates while there is a Kinesis Data Streams producer or consumer application writing to or reading data from the stream\. 

The provisioned mode is suited for predictable traffic with capacity requirements that are easy to forecast\. You can use the provisioned mode if you want fine\-grained control over how data is distributed across shards\. 

With the provisioned mode, you must specify the number of shards for the data stream\. To determine the size of a data stream with the provisioned mode, you need the following input values:
+ The average size of the data record written to the stream in kilobytes \(KB\), rounded up to the nearest 1 KB \(`average_data_size_in_KB`\)\.
+ The number of data records written to and read from the stream per second \(`records_per_second`\)\.
+ The number of consumers, which are Kinesis Data Streams applications that consume data concurrently and independently from the stream \(`number_of_consumers`\)\.
+ The incoming write bandwidth in KB \(`incoming_write_bandwidth_in_KB`\), which is equal to the `average_data_size_in_KB` multiplied by the `records_per_second`\.
+ The outgoing read bandwidth in KB \(`outgoing_read_bandwidth_in_KB`\), which is equal to the `incoming_write_bandwidth_in_KB` multiplied by the `number_of_consumers`\.

You can calculate the number of shards \(`number_of_shards`\) that your stream needs by using the input values in the following formula\.

```
number_of_shards = max(incoming_write_bandwidth_in_KiB/1024, outgoing_read_bandwidth_in_KiB/2048)
```

You may still experience read and write throughput exceptions in the provisioned mode if you don't configure your data stream to handle your peak throughput\. In this case, you must manually scale your data stream to accommodate your data traffic\. 

You may also experience read and write exceptions if you're using a partition key that leads to uneven data distribution and the records assigned to a shard exceed its limits\. To resolve this issue in the provisioned mode, identify such shards and manually split them to better accommodate your traffic\. For more information, see [Resharding a Stream](https://docs.aws.amazon.com/streams/latest/dev/kinesis-using-sdk-java-resharding.html)\. 

## Switching Between Capacity Modes<a name="switchingmodes"></a>

You can switch the capacity mode of your data stream from on\-demand to provisioned, or from provisioned to on\-demand\. For each data stream in your AWS account, you can switch between the on\-demand and provisioned capacity modes twice within 24 hours\.

Switching between capacity modes of a data stream does not cause any disruptions to your applications that use this data stream\. You can continue writing to and reading from this data stream\. As you are switching between capacity modes, either from on\-demand to provisioned or from provisioned to on\-demand, the status of the stream is set to *Updating*\. You must wait for the data stream status to get to *Active* before you can modify its properties again\.

When you switch from provisioned to on\-demand capacity mode, your data stream initially retains whatever shard count it had before the transition, and from this point on, Kinesis Data Streams monitors your data traffic and scales the shard count of this on\-demand data stream depending on your write throughput\. 

When you switch from on\-demand to provisioned mode, your data stream also initially retains whatever shard count it had before the transition, but from this point on, you are responsible for monitoring and adjusting the shard count of this data stream to properly accomodate your write throughput\.