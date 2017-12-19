# Retrieving Shards from a Stream<a name="kinesis-using-sdk-java-retrieve-shards"></a>

The response object returned by the `describeStream` method enables you to retrieve information about the shards that comprise the stream\. To retrieve the shards, call the `getShards` method on this object\. This method might not return all the shards from the stream in a single call\. In the following code, we check the `getHasMoreShards` method on `getStreamDescription` to see if there are additional shards that were not returned\. If so, that is, if this method returns `true`, we continue to call `getShards` in a loop, adding each new batch of returned shards to our list of shards\. The loop exits when `getHasMoreShards` returns `false`; that is, all shards have been returned\. Note that `getShards` does not return shards that are in the `EXPIRED` state\. For more information about shard states, including the `EXPIRED` state, see [Data Routing, Data Persistence, and Shard State after a Reshard](kinesis-using-sdk-java-after-resharding.md#kinesis-using-sdk-java-resharding-data-routing)\. 

```
DescribeStreamRequest describeStreamRequest = new DescribeStreamRequest();
describeStreamRequest.setStreamName( myStreamName );
List<Shard> shards = new ArrayList<>();
String exclusiveStartShardId = null;
do {
    describeStreamRequest.setExclusiveStartShardId( exclusiveStartShardId );
    DescribeStreamResult describeStreamResult = client.describeStream( describeStreamRequest );
    shards.addAll( describeStreamResult.getStreamDescription().getShards() );
    if (describeStreamResult.getStreamDescription().getHasMoreShards() && shards.size() > 0) {
        exclusiveStartShardId = shards.get(shards.size() - 1).getShardId();
    } else {
        exclusiveStartShardId = null;
    }
} while ( exclusiveStartShardId != null );
```