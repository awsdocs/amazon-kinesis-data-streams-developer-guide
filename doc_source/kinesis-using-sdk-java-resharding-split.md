# Splitting a Shard<a name="kinesis-using-sdk-java-resharding-split"></a>

To split a shard in Amazon Kinesis Data Streams, you need to specify how hash key values from the parent shard should be redistributed to the child shards\. When you add a data record to a stream, it is assigned to a shard based on a hash key value\. The hash key value is the [MD5](http://en.wikipedia.org/wiki/MD5) hash of the partition key that you specify for the data record at the time that you add the data record to the stream\. Data records that have the same partition key also have the same hash key value\.

The possible hash key values for a given shard constitute a set of ordered contiguous non\-negative integers\. This range of possible hash key values is given by the following: 

```
shard.getHashKeyRange().getStartingHashKey();
shard.getHashKeyRange().getEndingHashKey();
```

When you split the shard, you specify a value in this range\. That hash key value and all higher hash key values are distributed to one of the child shards\. All the lower hash key values are distributed to the other child shard\. 

The following code demonstrates a shard split operation that redistributes the hash keys evenly between each of the child shards, essentially splitting the parent shard in half\. This is just one possible way of dividing the parent shard\. You could, for example, split the shard so that the lower one\-third of the keys from the parent go to one child shard and the upper two\-thirds of the keys go to the other child shard\. However, for many applications, splitting shards in half is an effective approach\. 

The code assumes that `myStreamName` holds the name of your stream and the object variable `shard` holds the shard to split\. Begin by instantiating a new `splitShardRequest` object and setting the stream name and shard ID\.

```
SplitShardRequest splitShardRequest = new SplitShardRequest();
splitShardRequest.setStreamName(myStreamName);
splitShardRequest.setShardToSplit(shard.getShardId());
```

Determine the hash key value that is half\-way between the lowest and highest values in the shard\. This is the starting hash key value for the child shard that will contain the upper half of the hash keys from the parent shard\. Specify this value in the `setNewStartingHashKey` method\. You need specify only this value\. Kinesis Data Streams automatically distributes the hash keys below this value to the other child shard that is created by the split\. The last step is to call the `splitShard` method on the Kinesis Data Streams client\.

```
BigInteger startingHashKey = new BigInteger(shard.getHashKeyRange().getStartingHashKey());
BigInteger endingHashKey   = new BigInteger(shard.getHashKeyRange().getEndingHashKey());
String newStartingHashKey  = startingHashKey.add(endingHashKey).divide(new BigInteger("2")).toString();

splitShardRequest.setNewStartingHashKey(newStartingHashKey);
client.splitShard(splitShardRequest);
```

The first step after this procedure is shown in [Waiting for a Stream to Become Active Again](kinesis-using-sdk-java-after-resharding.md#kinesis-using-sdk-java-resharding-wait-until-active)\. 