# Step 4: Implement the Producer<a name="tutorial-stock-data-kplkcl-producer"></a>

The application in the [Tutorial: Process Real\-Time Stock Data Using KPL and KCL 1\.x[Tutorial: Process Real\-Time Stock Data Using KPL and KCL 1\.x](tutorial-stock-data-kplkcl.md)](tutorial-stock-data-kplkcl.md) uses the real\-world scenario of stock market trade monitoring\. The following principles briefly explain how this scenario maps to the producer and supporting code structure\.

Refer to the source code and review the following information\.

**StockTrade class**  
An individual stock trade is represented by an instance of the `StockTrade` class\. This instance contains attributes such as the ticker symbol, price, number of shares, the type of the trade \(buy or sell\), and an ID uniquely identifying the trade\. This class is implemented for you\.

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
`StockTradeGenerator` has a method called `getRandomTrade()` that returns a new randomly generated stock trade every time it is invoked\. This class is implemented for you\.

**StockTradesWriter class**  
The `main` method of the producer, `StockTradesWriter` continuously retrieves a random trade and then sends it to Kinesis Data Streams by performing the following tasks:  

1. Reads the stream name and Region name as input\.

1. Creates an `AmazonKinesisClientBuilder`\.

1. Uses the client builder to set the Region, credentials, and client configuration\.

1. Builds an `AmazonKinesis` client using the client builder\.

1. Checks that the stream exists and is active \(if not, it exits with an error\)\.

1. In a continuous loop, calls the `StockTradeGenerator.getRandomTrade()` method and then the `sendStockTrade` method to send the trade to the stream every 100 milliseconds\.
The `sendStockTrade` method of the `StockTradesWriter` class has the following code:  

```
private static void sendStockTrade(StockTrade trade, AmazonKinesis kinesisClient, String streamName) {
    byte[] bytes = trade.toJsonAsBytes();
    // The bytes could be null if there is an issue with the JSON serialization by the Jackson JSON library.
    if (bytes == null) {
        LOG.warn("Could not get JSON bytes for stock trade");
        return;
    }
    
    LOG.info("Putting trade: " + trade.toString());
    PutRecordRequest putRecord = new PutRecordRequest();
    putRecord.setStreamName(streamName);
    // We use the ticker symbol as the partition key, explained in the Supplemental Information section below.
    putRecord.setPartitionKey(trade.getTickerSymbol());
    putRecord.setData(ByteBuffer.wrap(bytes));

    try {
        kinesisClient.putRecord(putRecord);
    } catch (AmazonClientException ex) {
        LOG.warn("Error sending record to Amazon Kinesis.", ex);
    }
}
```

Refer to the following code breakdown:
+ The `PutRecord` API expects a byte array, and you need to convert `trade` to JSON format\. This single line of code performs that operation:

  ```
  byte[] bytes = trade.toJsonAsBytes();
  ```
+ Before you can send the trade, you create a new `PutRecordRequest` instance \(called `putRecord` in this case\):

  ```
  PutRecordRequest putRecord = new PutRecordRequest();
  ```

  Each `PutRecord` call requires the stream name, partition key, and data blob\. The following code populates these fields in the `putRecord` object using its `setXxxx()` methods:

  ```
  putRecord.setStreamName(streamName);
  putRecord.setPartitionKey(trade.getTickerSymbol());
  putRecord.setData(ByteBuffer.wrap(bytes));
  ```

  The example uses a stock ticket as a partition key, which maps the record to a specific shard\. In practice, you should have hundreds or thousands of partition keys per shard such that records are evenly dispersed across your stream\. For more information about how to add data to a stream, see [Adding Data to a Stream](developing-producers-with-sdk.md#kinesis-using-sdk-java-add-data-to-stream)\.

  Now `putRecord` is ready to send to the client \(the `put` operation\):

  ```
  kinesisClient.putRecord(putRecord);
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
         kinesisClient.putRecord(putRecord);
  } catch (AmazonClientException ex) {
         LOG.warn("Error sending record to Amazon Kinesis.", ex);
  }
  ```

  This is because a Kinesis Data Streams `put` operation can fail because of a network error, or due to the stream reaching its throughput limits and getting throttled\. We recommend carefully considering your retry policy for `put` operations to avoid data loss, such using as a simple retry\. 
+ Status logging is helpful but optional:

  ```
  LOG.info("Putting trade: " + trade.toString());
  ```
The producer shown here uses the Kinesis Data Streams API single record functionality, `PutRecord`\. In practice, if an individual producer generates many records, it is often more efficient to use the multiple records functionality of `PutRecords` and send batches of records at a time\. For more information, see [Adding Data to a Stream](developing-producers-with-sdk.md#kinesis-using-sdk-java-add-data-to-stream)\.

**To run the producer**

1. Verify that the access key and secret key pair retrieved earlier \(when creating the IAM user\) are saved in the file `~/.aws/credentials`\. 

1. Run the `StockTradeWriter` class with the following arguments:

   ```
   StockTradeStream us-west-2
   ```

   If you created your stream in a Region other than `us-west-2`, you have to specify that Region here instead\.

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

Your stock trade stream is now being ingested by Kinesis Data Streams\.

## Next Steps<a name="tutorial-stock-data-kplkcl-producer-next"></a>

[Step 5: Implement the Consumer](tutorial-stock-data-kplkcl-consumer.md)