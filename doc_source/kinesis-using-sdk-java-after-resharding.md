# After Resharding<a name="kinesis-using-sdk-java-after-resharding"></a>

After any kind of resharding procedure in Amazon Kinesis Data Streams, and before normal record processing resumes, other procedures and considerations are required\. The following sections describe these\.

**Topics**
+ [Waiting for a Stream to Become Active Again](#kinesis-using-sdk-java-resharding-wait-until-active)
+ [Data Routing, Data Persistence, and Shard State after a Reshard](#kinesis-using-sdk-java-resharding-data-routing)

## Waiting for a Stream to Become Active Again<a name="kinesis-using-sdk-java-resharding-wait-until-active"></a>

After you call a resharding operation, either `splitShard` or `mergeShards`, you need to wait for the stream to become active again\. The code to use is the same as when you wait for a stream to become active after [creating a stream](kinesis-using-sdk-java-create-stream.md)\. That code is as follows:

```
DescribeStreamRequest describeStreamRequest = new DescribeStreamRequest();
describeStreamRequest.setStreamName( myStreamName );

long startTime = System.currentTimeMillis();
long endTime = startTime + ( 10 * 60 * 1000 );
while ( System.currentTimeMillis() < endTime ) 
{
  try {
    Thread.sleep(20 * 1000);
  } 
  catch ( Exception e ) {}
  
  try {
    DescribeStreamResult describeStreamResponse = client.describeStream( describeStreamRequest );
    String streamStatus = describeStreamResponse.getStreamDescription().getStreamStatus();
    if ( streamStatus.equals( "ACTIVE" ) ) {
      break;
    }
   //
    // sleep for one second
    //
    try {
      Thread.sleep( 1000 );
    }
    catch ( Exception e ) {}
  }
  catch ( ResourceNotFoundException e ) {}
}
if ( System.currentTimeMillis() >= endTime ) 
{
  throw new RuntimeException( "Stream " + myStreamName + " never went active" );
}
```

## Data Routing, Data Persistence, and Shard State after a Reshard<a name="kinesis-using-sdk-java-resharding-data-routing"></a>

Kinesis Data Streams is a real\-time data streaming service, which is to say that your applications should assume that data is flowing continuously through the shards in your stream\. When you reshard, data records that were flowing to the parent shards are re\-routed to flow to the child shards based on the hash key values that the data\-record partition keys map to\. However, any data records that were in the parent shards before the reshard remain in those shards\. In other words, the parent shards do not disappear when the reshard occurs\. They persist along with the data they contained before the reshard\. The data records in the parent shards are accessible using the [`getShardIterator` and `getRecords`](developing-consumers-with-sdk.md#kinesis-using-sdk-java-get-data) operations in the Kinesis Data Streams API, or through the Kinesis Client Library\.

**Note**  
Data records are accessible from the time they are added to the stream to the current retention period\. This holds true regardless of any changes to the shards in the stream during that time period\. For more information about a stream’s retention period, see [Changing the Data Retention Period](kinesis-extended-retention.md)\.

In the process of resharding, a parent shard transitions from an `OPEN` state to a `CLOSED` state to an `EXPIRED` state\. 
+  **OPEN**: Before a reshard operation, a parent shard is in the `OPEN` state, which means that data records can be both added to the shard and retrieved from the shard\.
+  **CLOSED**: After a reshard operation, the parent shard transitions to a `CLOSED` state\. This means that data records are no longer added to the shard\. Data records that would have been added to this shard are now added to a child shard instead\. However, data records can still be retrieved from the shard for a limited time\. 
+  **EXPIRED**: After the stream's retention period has expired, all the data records in the parent shard have expired and are no longer accessible\. At this point, the shard itself transitions to an `EXPIRED` state\. Calls to `getStreamDescription().getShards` to enumerate the shards in the stream do not include `EXPIRED` shards in the list shards returned\. For more information about a stream’s retention period, see [Changing the Data Retention Period](kinesis-extended-retention.md)\.

After the reshard has occurred and the stream is again in an `ACTIVE` state, you could immediately begin to read data from the child shards\. However, the parent shards that remain after the reshard could still contain data that you haven't read yet that was added to the stream before the reshard\. If you read data from the child shards before having read all data from the parent shards, you could read data for a particular hash key out of the order given by the data records' sequence numbers\. Therefore, assuming that the order of the data is important, you should, after a reshard, always continue to read data from the parent shards until it is exhausted\. Only then should you begin reading data from the child shards\. When `getRecordsResult.getNextShardIterator` returns `null`, it indicates that you have read all the data in the parent shard\. If you are reading data using the Kinesis Client Library, the library ensures that you receive the data in order even if a reshard occurs\.