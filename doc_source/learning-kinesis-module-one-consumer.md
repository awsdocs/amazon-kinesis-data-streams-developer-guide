# Step 5: Implement the Consumer<a name="learning-kinesis-module-one-consumer"></a>

The consumer application you are developing continuously processes the stock trades stream that you created in [Step 4: Implement the Producer](learning-kinesis-module-one-producer.md), and outputs the most popular stocks being bought and sold every minute\. The application is built on top of the KCL, which does a lot of the heavy lifting common to consumer apps\. For more information, see [Developing Amazon Kinesis Streams Consumers Using the Kinesis Client Library](developing-consumers-with-kcl.md)\. 

Refer to the source code and review the following information\.

**StockTradesProcessor class**  
Main class of the consumer, provided for you, which performs the following tasks:  

+ Read the application, stream, and Region names, passed in as arguments\.

+ Read credentials from `~/.aws/credentials`\.

+ Create a `RecordProcessorFactory` instance that serves instances of `RecordProcessor`, implemented by a `StockTradeRecordProcessor` instance\.

+ Create a KCL worker with the `RecordProcessorFactory` instance and a standard configuration including the stream name, credentials, and application name\. 

+ The worker creates a new thread for each shard \(assigned to this consumer instance\), which continuously loops to read records from Kinesis Streams and then invokes the `RecordProcessor` instance to process each batch of records received\.

**StockTradeRecordProcessor class**  
Implementation of the `RecordProcessor` instance, which in turn implements three required methods: `initialize`, `processRecords`, and `shutdown`\.  
As the names suggest, `initialize` and `shutdown` are used by KCL to let the record processor know when it should be ready to start receiving records and when it should expect to stop receiving records, respectively, so it can do any application\-specific setup and termination tasks\. The code for these is provided for you\. The main processing happens in the `processRecords` method, which in turn uses `processRecord` for each record\. This latter method is provided as mostly empty skeleton code for you to implement in the next step, where it is explained further\.  
Also of note is the implementation of support methods for `processRecord`: `reportStats`, and `resetStats`, which are empty in the original source code\.  
The `processRecords` method is implemented for you, and performs the following steps:  

+  For each record passed in, call `processRecord` on it\.

+ If at least 1 minute has elapsed since the last report, call `reportStats()` which prints out the latest stats, and then `resetStats()` which clears the stats so that the next interval includes only new records\.

+ Set the next reporting time\.

+ If at least 1 minute has elapsed since the last checkpoint, call `checkpoint()`\. 

+ Set the next checkpointing time\.
This method uses 60\-second intervals for the reporting and checkpointing rate\. For more information about checkpointing, see [Additional Information About the Consumer](#learning-kinesis-module-one-consumer-supplement)\.

**StockStats class**  
This class provides data retention and statistics tracking for the most popular stocks over time\. This code is provided for you and contains the following methods:  

+ `addStockTrade(StockTrade)`: Injects the given `StockTrade` into the running statistics\.

+ `toString()`: Returns the statistics in a formatted string\.
The way this class keeps track of the most popular stock is that it keeps a running count of the total number of trades for each stock and the maximum count\. It keeps these counts updated whenever a stock trade arrives\.

Add code to the methods of the `StockTradeRecordProcessor` class, as shown in the following steps\.

**To implement the consumer**

1. Implement the `processRecord` method by instantiating a correctly sized `StockTrade` object and adding the record data to it, logging a warning if there's a problem\.

   ```
   StockTrade trade = StockTrade.fromJsonAsBytes(record.getData().array());
   if (trade == null) {
       LOG.warn("Skipping record. Unable to parse record into StockTrade. Partition Key: " + record.getPartitionKey());
       return;
   }
   stockStats.addStockTrade(trade);
   ```

1. Implement a simple `reportStats` method\. Feel free to modify the output format to your preferences\.

   ```
   System.out.println("****** Shard " + kinesisShardId + " stats for last 1 minute ******\n" +
                      stockStats + "\n" +
                      "****************************************************************\n");
   ```

1. Finally, implement the `resetStats` method, which creates a new `stockStats` instance\.

   ```
   stockStats = new StockStats();
   ```

**To run the consumer**

1. Run the producer you wrote in the previous module to inject simulated stock trade records into your stream\.

1. Verify that the access key and secret key pair retrieved earlier \(when creating the IAM user\) are saved in the file `~/.aws/credentials` \. 

1. Run the `StockTradesProcessor` class with the following arguments:

   ```
   StockTradesProcessor StockTradeStream us-west-2
   ```

   Note that if you created your stream in a Region other than `us-west-2`, you have to specify that Region here instead\.

After a minute, you should see output like the following, refreshed every minute thereafter:

```
  ****** Shard shardId-000000000001 stats for last 1 minute ******
  Most popular stock being bought: WMT, 27 buys.
  Most popular stock being sold: PTR, 14 sells.
  ****************************************************************
```

## Additional Information About the Consumer<a name="learning-kinesis-module-one-consumer-supplement"></a>

If you are familiar with the advantages of the KCL, discussed in [Developing Amazon Kinesis Streams Consumers Using the Kinesis Client Library](developing-consumers-with-kcl.md) and elsewhere, you may be wondering why you should use it here\. Although you use only a single shard stream and a single consumer instance to process it, it is still easier to implement the consumer with the KCL\. Compare the code implementation steps in the producer section to the consumer, and you can see the comparative ease of implementing a consumer\. This is largely due to the services that the KCL provides\.

In this application, you focus on implementing a record processor class that can process individual records\. You don’t have to worry about how the records are fetched from Kinesis Streams; The KCL fetches the records and invoke the record processor whenever there are new records available\. Also, you don’t have to worry about how many shards and consumer instances there are; if the stream is scaled up, you don’t have to rewrite your application to handle more than one shard or one consumer instance\.

The term *checkpointing* means recording the point in the stream up to the data records that have been consumed and processed thus far, so that if the application crashes, the stream is read from that point and not from the beginning of the stream\. The subject of checkpointing and the various design patterns and best practices for it are outside the scope of this chapter, but is something you may be confronted with in production environments\.

As you learned in [Step 4: Implement the Producer](learning-kinesis-module-one-producer.md), the `put` operations in the Kinesis Streams API take a *partition key* as input\. A partition key is used by Kinesis Streams as a mechanism to split records across multiple shards \(when there is more than one shard in the stream\)\. The same partition key always routes to the same shard\. This allows the consumer that processes a particular shard to be designed with the assumption that records with the same partition key would only be sent to that consumer, and no records with the same partition key would end up at any other consumer\. Therefore, a consumer's worker can aggregate all records with the same partition key without worrying that it might be missing needed data\.

In this application, the consumer's processing of records is not intensive, so you can use one shard and do the processing in the same thread as the KCL thread\. In practice, however, consider first scaling up the number of shards, as the next module in this learning series demonstrates\. In some cases you may want to switch processing to a different thread, or use a thread pool if your record processing is expected to be intensive\. In this way, the KCL can fetch new records more quickly while the other threads can process the records in parallel\. Note that multithreaded design is not trivial and should be approached with advanced techniques, so increasing your shard count is usually the most effective and easiest way to scale up\.
