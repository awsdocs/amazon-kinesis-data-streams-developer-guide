# Step 5: Implement the Consumer<a name="tutorial-stock-data-kplkcl2-consumer"></a>

The consumer application in this tutorial continuously processes the stock trades in your data stream\. It then outputs the most popular stocks being bought and sold every minute\. The application is built on top of the Kinesis Client Library \(KCL\), which does much of the heavy lifting common to consumer apps\. For more information, see [Using the Kinesis Client Library](shared-throughput-kcl-consumers.md)\. 

Refer to the source code and review the following information\.

**StockTradesProcessor class**  
Main class of the consumer, provided for you, which performs the following tasks:  
+ Reads the application, data stream, and region names, passed in as arguments\.
+ Creates a `KinesisAsyncClient` instance with the region name\.
+ Creates a `StockTradeRecordProcessorFactory` instance that serves instances of `ShardRecordProcessor`, implemented by a `StockTradeRecordProcessor` instance\. 
+ Creates a `ConfigsBuilder` instance with the `KinesisAsyncClient`, `StreamName`, `ApplicationName` and `StockTradeRecordProcessorFactory` instance\. This is useful for creating all configurations with default values\.
+ Creates a KCL scheduler \(previously, in KCL versions 1\.x it was known as the KCL worker\) with the `ConfigsBuilder` instance\. 
+ The scheduler creates a new thread for each shard \(assigned to this consumer instance\), which continuously loops to read records from the data stream\. It then invokes the `StockTradeRecordProcessor` instance to process each batch of records received\. 

**StockTradeRecordProcessor class**  
Implementation of the `StockTradeRecordProcessor` instance, which in turn implements five required methods: `initialize`, `processRecords`, `leaseLost`, `shardEnded`, and `shutdownRequested`\.   
The `initialize` and `shutdownRequested` methods are used by the KCL to let the record processor know when it should be ready to start receiving records and when it should expect to stop receiving records, respectively, so it can perform any application\-specific setup and termination tasks\. `leaseLost` and `shardEnded` are used to implement any logic for what to do when a lease is lost or a processing has reached the end of a shard\. In this example, we simply log messages indicating these events\.   
The code for these methods is provided for you\. The main processing happens in the `processRecords` method, which in turn uses `processRecord` for each record\. This latter method is provided as the mostly empty skeleton code for you to implement in the next step, where it is explained in greater detail\.   
Also of note is the implementation of the support methods for `processRecord`: `reportStats`, and `resetStats`, which are empty in the original source code\.   
The `processRecords` method is implemented for you, and performs the following steps:  
+ For each record passed in, it calls `processRecord` on it\. 
+ If at least 1 minute has elapsed since the last report, calls `reportStats()`, which prints out the latest stats, and then `resetStats()` which clears the stats so that the next interval includes only new records\.
+ Sets the next reporting time\.
+ If at least 1 minute has elapsed since the last checkpoint, calls `checkpoint()`\. 
+ Sets the next checkpointing time\.
This method uses 60\-second intervals for the reporting and checkpointing rate\. For more information about checkpointing, see ??? 

**StockStats class**  
This class provides data retention and statistics tracking for the most popular stocks over time\. This code is provided for you and contains the following methods:  
+ `addStockTrade(StockTrade)`: injects the given `StockTrade` into the running statistics\.
+ `toString()`: returns the statistics in a formatted string\.
This class keeps track of the most popular stock by keeping a running count of the total number of trades for each stock and the maximum count\. It updates these counts whenever a stock trade arrives\.

Add code to the methods of the `StockTradeRecordProcessor` class, as shown in the following steps\. 

**To implement the consumer**

1. Implement the `processRecord` method by instantiating a correctly sized `StockTrade` object and adding the record data to it, logging a warning if there's a problem\. 

   ```
   byte[] arr = new byte[record.data().remaining()];
   record.data().get(arr);
   StockTrade trade = StockTrade.fromJsonAsBytes(arr);
       if (trade == null) {
           log.warn("Skipping record. Unable to parse record into StockTrade. Partition Key: " + record.partitionKey());
           return;
           }
   stockStats.addStockTrade(trade);
   ```

1. Implement a simple `reportStats` method\. Feel free to modify the output format to suit your preferences\. 

   ```
   System.out.println("****** Shard " + kinesisShardId + " stats for last 1 minute ******\n" +
   stockStats + "\n" +
   "****************************************************************\n");
   ```

1. Implement the `resetStats` method, which creates a new `stockStats` instance\. 

   ```
   stockStats = new StockStats();
   ```

1. Implement the following methods required by `ShardRecordProcessor` interface

   ```
   @Override
   public void leaseLost(LeaseLostInput leaseLostInput) {
       log.info("Lost lease, so terminating.");
   }
   
   @Override
   public void shardEnded(ShardEndedInput shardEndedInput) {
       try {
           log.info("Reached shard end checkpointing.");
           shardEndedInput.checkpointer().checkpoint();
       } catch (ShutdownException | InvalidStateException e) {
           log.error("Exception while checkpointing at shard end. Giving up.", e);
       }
   }
   
   @Override
   public void shutdownRequested(ShutdownRequestedInput shutdownRequestedInput) {
       log.info("Scheduler is shutting down, checkpointing.");
       checkpoint(shutdownRequestedInput.checkpointer());
   }
   
   private void checkpoint(RecordProcessorCheckpointer checkpointer) {
       log.info("Checkpointing shard " + kinesisShardId);
       try {
           checkpointer.checkpoint();
       } catch (ShutdownException se) {
           // Ignore checkpoint if the processor instance has been shutdown (fail over).
           log.info("Caught shutdown exception, skipping checkpoint.", se);
       } catch (ThrottlingException e) {
           // Skip checkpoint when throttled. In practice, consider a backoff and retry policy.
           log.error("Caught throttling exception, skipping checkpoint.", e);
       } catch (InvalidStateException e) {
           // This indicates an issue with the DynamoDB table (check for table, provisioned IOPS).
           log.error("Cannot save checkpoint to the DynamoDB table used by the Amazon Kinesis Client Library.", e);
       }
   }
   ```

**To run the consumer**

1. Run the producer that you wrote in [[Step 4: Implement the Producer](tutorial-stock-data-kplkcl2-producer.md)Step 4: Implement the Producer](tutorial-stock-data-kplkcl2-producer.md) to inject simulated stock trade records into your stream\.

1. Verify that the access key and secret key pair retrieved earlier \(when creating the IAM user\) are saved in the file `~/.aws/credentials`\. 

1. Run the `StockTradesProcessor` class with the following arguments:

   ```
   StockTradesProcessor StockTradeStream us-west-2
   ```

   Note that if you created your stream in a region other than `us-west-2`, you have to specify that region here instead\.

After a minute, you should see output like the following, refreshed every minute thereafter:

```
  
  ****** Shard shardId-000000000001 stats for last 1 minute ******
  Most popular stock being bought: WMT, 27 buys.
  Most popular stock being sold: PTR, 14 sells.
  ****************************************************************
```

## Next Steps<a name="tutorial-stock-data-kplkcl2-consumer-next"></a>

[Step 6: \(Optional\) Extending the Consumer](tutorial-stock-data-kplkcl2-consumer-extension.md)