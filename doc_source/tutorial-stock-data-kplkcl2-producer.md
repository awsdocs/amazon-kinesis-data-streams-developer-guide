# Step 4: Implement the Producer<a name="tutorial-stock-data-kplkcl2-producer"></a>

This tutorial uses the real\-world scenario of stock market trade monitoring\. The following principles briefly explain how this scenario maps to the producer and its supporting code structure\.

Refer to the [source code](https://github.com/aws-samples/amazon-kinesis-learning ) and review the following information\.

**StockTrade class**  
An individual stock trade is represented by an instance of the StockTrade class\. This instance contains attributes such as the ticker symbol, price, number of shares, the type of the trade \(buy or sell\), and an ID uniquely identifying the trade\. This class is implemented for you\. 

**Stream record**  
A stream is a sequence of records\. A record is a serialization of a `StockTrade` instance in JSON format\. For example:   

```
{
  "tickerSymbol": "AMZN", 
  "tradeType": "BUY", 
  "price": 395.87,
  "quantity": 16, 
  "id": 3567129045
}
```

**StockTradeGenerator class**  
StockTradeGenerator has a method called `getRandomTrade()` that returns a new randomly generated stock trade every time it is invoked\. This class is implemented for you\.

**StockTradesWriter class**  
The `main` method of the producer, StockTradesWriter continuously retrieves a random trade and then sends it to Kinesis Data Streams by performing the following tasks:  

1. Reads the data stream name and region name as input\.

1. Uses the `KinesisAsyncClientBuilder` to set the region, credentials, and client configuration\. 

1. Checks that the stream exists and is active \(if not, it exits with an error\)\. 

1. In a continuous loop, calls the `StockTradeGenerator.getRandomTrade()` method and then the `sendStockTrade` method to send the trade to the stream every 100 milliseconds\. 
The `sendStockTrade` method of the `StockTradesWriter` class has the following code:   

```
private static void sendStockTrade(StockTrade trade, KinesisAsyncClient kinesisClient,
            String streamName) {
        byte[] bytes = trade.toJsonAsBytes();
        // The bytes could be null if there is an issue with the JSON serialization by the Jackson JSON library.
        if (bytes == null) {
            LOG.warn("Could not get JSON bytes for stock trade");
            return;
        }

        LOG.info("Putting trade: " + trade.toString());
        PutRecordRequest request = PutRecordRequest.builder()
                .partitionKey(trade.getTickerSymbol()) // We use the ticker symbol as the partition key, explained in the Supplemental Information section below.
                .streamName(streamName)
                .data(SdkBytes.fromByteArray(bytes))
                .build();
        try {
            kinesisClient.putRecord(request).get();
        } catch (InterruptedException e) {
            LOG.info("Interrupted, assuming shutdown.");
        } catch (ExecutionException e) {
            LOG.error("Exception while sending data to Kinesis. Will try again next cycle.", e);
        }
    }
```

Refer to the following code breakdown:
+ The `PutRecord` API expects a byte array, and you need to convert trade to JSON format\. This single line of code performs that operation: 

  ```
  byte[] bytes = trade.toJsonAsBytes();
  ```
+ Before you can send the trade, you create a new `PutRecordRequest` instance \(called request in this case\)\. Each `request` requires the stream name, partition key, and a data blob\. 

  ```
  PutPutRecordRequest request = PutRecordRequest.builder()
      .partitionKey(trade.getTickerSymbol()) // We use the ticker symbol as the partition key, explained in the Supplemental Information section below.
      .streamName(streamName)
      .data(SdkBytes.fromByteArray(bytes))
      .build();
  ```

  The example uses a stock ticket as a partition key, which maps the record to a specific shard\. In practice, you should have hundreds or thousands of partition keys per shard such that records are evenly dispersed across your stream\. For more information about how to add data to a stream, see [Writing Data to Amazon Kinesis Data Streams](building-producers.md)\.

  Now `request` is ready to send to the client \(the put operation\): 

  ```
     kinesisClient.putRecord(request).get();
  ```
+ Error checking and logging are always useful additions\. This code logs error conditions: 

  ```
  if (bytes == null) {
      LOG.warn("Could not get JSON bytes for stock trade");
      return;
  }
  ```

  Add the try/catch block around the `put` operation: 

  ```
  try {
   	kinesisClient.putRecord(request).get();
  } catch (InterruptedException e) {
              LOG.info("Interrupted, assuming shutdown.");
  } catch (ExecutionException e) {
              LOG.error("Exception while sending data to Kinesis. Will try again next cycle.", e);
  }
  ```

  This is because a Kinesis Data Streams put operation can fail because of a network error, or due to the data stream reaching its throughput limits and getting throttled\. It is recommended that you carefully consider your retry policy for `put` operations to avoid data loss, such using as a simple retry\. 
+ Status logging is helpful but optional:

  ```
  LOG.info("Putting trade: " + trade.toString());
  ```
The producer shown here uses the Kinesis Data Streams API single record functionality, `PutRecord`\. In practice, if an individual producer generates many records, it is often more efficient to use the multiple records functionality of `PutRecords` and send batches of records at a time\. For more information, see [Writing Data to Amazon Kinesis Data Streams](building-producers.md)\.

**To run the producer**

1. Verify that the access key and secret key pair retrieved in [Step 2: Create an IAM Policy and User](tutorial-stock-data-kplkcl2-iam.md) are saved in the file `~/.aws/credentials`\. 

1. Run the `StockTradeWriter` class with the following arguments:

   ```
   StockTradeStream us-west-2
   ```

   If you created your stream in a region other than `us-west-2`, you have to specify that region here instead\.

You should see output similar to the following:

```
Feb 16, 2015 3:53:00 PM  
com.amazonaws.services.kinesis.samples.stocktrades.writer.StockTradesWriter sendStockTrade
INFO: Putting trade: ID 8: SELL 996 shares of BUD for $124.18
Feb 16, 2015 3:53:00 PM 
com.amazonaws.services.kinesis.samples.stocktrades.writer.StockTradesWriter sendStockTrade
INFO: Putting trade: ID 9: BUY 159 shares of GE for $20.85
Feb 16, 2015 3:53:01 PM 
com.amazonaws.services.kinesis.samples.stocktrades.writer.StockTradesWriter sendStockTrade
INFO: Putting trade: ID 10: BUY 322 shares of WMT for $90.08
```

Your stock trades are now being ingested by Kinesis Data Streams\.

## Next Steps<a name="tutorial-stock-data-kplkcl2-producer-next"></a>

[Step 5: Implement the Consumer](tutorial-stock-data-kplkcl2-consumer.md)