# Consumer De\-aggregation<a name="kinesis-kpl-consumer-deaggregation"></a>

Beginning with release 1\.4\.0, the KCL supports automatic de\-aggregation of KPL user records\. Consumer application code written with previous versions of the KCL will compile without any modification after you update the KCL\. However, if KPL aggregation is being used on the producer side, there is a subtlety involving checkpointing: all subrecords within an aggregated record have the same sequence number, so additional data has to be stored with the checkpoint if you need to distinguish between subrecords\. This additional data is referred to as the *subsequence number*\.

## Migrating from Previous Versions of the KCL<a name="kinesis-kpl-consumer-deaggregation-migration"></a>

You are not required to change your existing calls to do checkpointing in conjunction with aggregation\. It is still guaranteed that you can retrieve all records successfully stored in Kinesis Data Streams\. The KCL now provides two new checkpoint operations to support particular use cases, described below\.

In the event that your existing code was written for the KCL prior to KPL support, and your checkpoint operation is called without arguments, it is equivalent to checkpointing the sequence number of the last KPL user record in the batch\. If your checkpoint operation is called with a sequence number string, it is equivalent to checkpointing the given sequence number of the batch along with the implicit subsequence number 0 \(zero\)\.

Calling the new KCL checkpoint operation `checkpoint()` without any arguments is semantically equivalent to checkpointing the sequence number of the last `Record` call in the batch, along with the implicit subsequence number 0 \(zero\)\. 

Calling the new KCL checkpoint operation `checkpoint(Record record)` is semantically equivalent to checkpointing the given `Record`’s sequence number along with the implicit subsequence number 0 \(zero\)\. If the `Record` call is actually a `UserRecord`, the `UserRecord` sequence number and subsequence number are checkpointed\. 

Calling the new KCL checkpoint operation `checkpoint(String sequenceNumber, long subSequenceNumber)` explicitly checkpoints the given sequence number along with the given subsequence number\. 

In any of these cases, after the checkpoint is stored in the Amazon DynamoDB checkpoint table, the KCL can correctly resume retrieving records even when the application crashes and restarts\. If more records are contained within the sequence, retrieval occurs starting with the next subsequence number record within the record with the most recently checkpointed sequence number\. If the most recent checkpoint included the very last subsequence number of the previous sequence number record, retrieval occurs starting with the record with the next sequence number\. 

The next section discusses details of sequence and subsequence checkpointing for consumers that need to avoid skipping and duplication of records\. If skipping \(or duplication\) of records when stopping and restarting your consumer’s record processing is not important, you can run your existing code with no modification\.

## KCL Extensions for KPL De\-aggregation<a name="kinesis-kpl-consumer-deaggregation-extensions"></a>

As previously discussed, KPL de\-aggregation can involve subsequence checkpointing\. To facilitate using subsequence checkpointing, a `UserRecord` class has been added to the KCL:

```
public class UserRecord extends Record {     
    public long getSubSequenceNumber() {
    /* ... */
    }      
    @Override 
    public int hashCode() {
    /* contract-satisfying implementation */ 
    }      
    @Override 
    public boolean equals(Object obj) {
    /* contract-satisfying implementation */ 
    } 
}
```

This class is now used instead of `Record`\. This does not break existing code because it is a subclass of `Record`\. The `UserRecord` class represents both actual subrecords and standard, non\-aggregated records\. Non\-aggregated records can be thought of as aggregated records with exactly one subrecord\.

In addition, two new operations are added to`IRecordProcessorCheckpointer`:

```
public void checkpoint(Record record); 
public void checkpoint(String sequenceNumber, long subSequenceNumber);
```

To begin using subsequence number checkpointing, you can perform the following conversion\. Change the following form code:

```
checkpointer.checkpoint(record.getSequenceNumber());
```

New form code:

```
checkpointer.checkpoint(record);
```

We recommend that you use the `checkpoint(Record record)` form for subsequence checkpointing\. However, if you are already storing `sequenceNumbers` in strings to use for checkpointing, you should now also store `subSequenceNumber`, as shown in the following example:

```
String sequenceNumber = record.getSequenceNumber(); 
long subSequenceNumber = ((UserRecord) record).getSubSequenceNumber();  // ... do other processing  
checkpointer.checkpoint(sequenceNumber, subSequenceNumber);
```

The cast from `Record`to`UserRecord` always succeeds because the implementation always uses `UserRecord` under the hood\. Unless there is a need to perform arithmetic on the sequence numbers, this approach is not recommended\.

While processing KPL user records, the KCL writes the subsequence number into Amazon DynamoDB as an extra field for each row\. Previous versions of the KCL used `AFTER_SEQUENCE_NUMBER` to fetch records when resuming checkpoints\. The current KCL with KPL support uses `AT_SEQUENCE_NUMBER` instead\. When the record at the checkpointed sequence number is retrieved, the checkpointed subsequence number is checked, and subrecords are dropped as appropriate \(which may be all of them, if the last subrecord is the one checkpointed\)\. Again, non\-aggregated records can be thought of as aggregated records with a single subrecord, so the same algorithm works for both aggregated and non\-aggregated records\.

## Using GetRecords Directly<a name="kinesis-kpl-consumer-deaggregation-getrecords"></a>

You can also choose not to use the KCL but instead invoke the API operation `GetRecords` directly to retrieve Kinesis Data Streams records\. To unpack these retrieved records into your original KPL user records, call one of the following static operations in `UserRecord.java`:

```
public static List<Record> deaggregate(List<Record> records)

public static List<UserRecord> deaggregate(List<UserRecord> records, BigInteger startingHashKey, BigInteger endingHashKey)
```

The first operation uses the default value `0` \(zero\) for `startingHashKey` and the default value `2^128 -1` for `endingHashKey`\.

Each of these operations de\-aggregates the given list of Kinesis Data Streams records into a list of KPL user records\. Any KPL user records whose explicit hash key or partition key falls outside the range of the `startingHashKey` \(inclusive\) and the `endingHashKey` \(inclusive\) are discarded from the returned list of records\.