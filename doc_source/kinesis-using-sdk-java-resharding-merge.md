# Merging Two Shards<a name="kinesis-using-sdk-java-resharding-merge"></a>

 A shard merge operation takes two specified shards and combines them into a single shard\. After the merge, the single child shard receives data for all hash key values covered by the two parent shards\. 

**Shard Adjacency**  
To merge two shards, the shards must be *adjacent*\. Two shards are considered adjacent if the union of the hash key ranges for the two shards forms a contiguous set with no gaps\. For example, suppose that you have two shards, one with a hash key range of 276\.\.\.381 and the other with a hash key range of 382\.\.\.454\. You could merge these two shards into a single shard that would have a hash key range of 276\.\.\.454\. 

To take another example, suppose that you have two shards, one with a hash key range of 276\.\.381 and the other with a hash key range of 455\.\.\.560\. You could not merge these two shards because there would be one or more shards between these two that cover the range 382\.\.454\. 

The set of all `OPEN` shards in a stream—as a group—always spans the entire range of MD5 hash key values\. For more information about shard states—such as `CLOSED`—see [Data Routing, Data Persistence, and Shard State after a Reshard](kinesis-using-sdk-java-after-resharding.md#kinesis-using-sdk-java-resharding-data-routing)\. 

To identify shards that are candidates for merging, you should filter out all shards that are in a `CLOSED` state\. Shards that are `OPEN`—that is, not `CLOSED`—have an ending sequence number of `null`\. You can test the ending sequence number for a shard using: 

```
if( null == shard.getSequenceNumberRange().getEndingSequenceNumber() ) 
{
  // Shard is OPEN, so it is a possible candidate to be merged.
}
```

After filtering out the closed shards, sort the remaining shards by the highest hash key value supported by each shard\. You can retrieve this value using: 

```
shard.getHashKeyRange().getEndingHashKey();
```

 If two shards are adjacent in this filtered, sorted list, they can be merged\. 

**Code for the Merge Operation**  
 The following code merges two shards\. The code assumes that `myStreamName` holds the name of your stream and the object variables `shard1` and `shard2` hold the two adjacent shards to merge\.

For the merge operation, begin by instantiating a new `mergeShardsRequest` object\. Specify the stream name with the `setStreamName` method\. Then specify the two shards to merge using the `setShardToMerge` and `setAdjacentShardToMerge` methods\. Finally, call the `mergeShards` method on the Kinesis Data Streams client to carry out the operation\.

```
MergeShardsRequest mergeShardsRequest = new MergeShardsRequest();
mergeShardsRequest.setStreamName(myStreamName);
mergeShardsRequest.setShardToMerge(shard1.getShardId());
mergeShardsRequest.setAdjacentShardToMerge(shard2.getShardId());
client.mergeShards(mergeShardsRequest);
```

The first step after this procedure is shown in [Waiting for a Stream to Become Active Again](kinesis-using-sdk-java-after-resharding.md#kinesis-using-sdk-java-resharding-wait-until-active)\.