# Low\-Latency Processing<a name="kinesis-low-latency"></a>

*Propagation delay* is defined as the end\-to\-end latency from the moment a record is written to the stream until it is read by a consumer application\. This delay varies depending upon a number of factors, but it is primarily affected by the polling interval of consumer applications\.

For most applications, we recommend polling each shard one time per second per application\. This enables you to have multiple consumer applications processing a stream concurrently without hitting Amazon Kinesis Data Streams limits of 5 `GetRecords` calls per second\. Additionally, processing larger batches of data tends to be more efficient at reducing network and other downstream latencies in your application\.

The KCL defaults are set to follow the best practice of polling every 1 second\. This default results in average propagation delays that are typically below 1 second\.

Kinesis Data Streams records are available to be read immediately after they are written\. There are some use cases that need to take advantage of this and require consuming data from the stream as soon as it is available\. You can significantly reduce the propagation delay by overriding the KCL default settings to poll more frequently, as shown in the following examples\.

Java KCL configuration code:

```
kinesisClientLibConfiguration = new
        KinesisClientLibConfiguration(applicationName,
        streamName,               
        credentialsProvider,
        workerId).withInitialPositionInStream(initialPositionInStream).withIdleTimeBetweenReadsInMillis(250);
```

Property file setting for Python and Ruby KCL:

```
idleTimeBetweenReadsInMillis = 250
```

**Note**  
Because Kinesis Data Streams has a limit of 5 `GetRecords` calls per second, per shard, setting the `idleTimeBetweenReadsInMillis` property lower than 200ms may result in your application observing the `ProvisionedThroughputExceededException` exception\. Too many of these exceptions can result in exponential back\-offs and thereby cause significant unexpected latencies in processing\. If you set this property to be at or just above 200 ms and have more than one processing application, you will experience similar throttling\.