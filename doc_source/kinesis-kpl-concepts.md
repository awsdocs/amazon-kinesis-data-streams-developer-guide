# Key Concepts<a name="kinesis-kpl-concepts"></a>

The following sections contain concepts and terminology necessary to understand and benefit from the KPL\.


+ [Records](#w3ab1c11b7b7c19b7)
+ [Batching](#w3ab1c11b7b7c19b9)
+ [Aggregation](#w3ab1c11b7b7c19c11)
+ [Collection](#w3ab1c11b7b7c19c13)

## Records<a name="w3ab1c11b7b7c19b7"></a>

In this guide, we distinguish between *KPL user records* and *Kinesis Streams records*\. When we use the term *record* without a qualifier, we refer to a *KPL user record*\. When we refer to a Kinesis Streams record, we will explicitly say *Kinesis Streams record*\.

A KPL user record is a blob of data that has particular meaning to the user\. Examples include a JSON blob representing a UI event on a web site, or a log entry from a web server\.

A Kinesis Streams record is an instance of the `Record` data structure defined by the Kinesis Streams service API\. It contains a partition key, sequence number, and a blob of data\. 

## Batching<a name="w3ab1c11b7b7c19b9"></a>

*Batching* refers to performing a single action on multiple items instead of repeatedly performing the action on each individual item\. 

In this context, the "item" is a record, and the action is sending it to Kinesis Streams\. In a non\-batching situation, you would place each record in a separate Kinesis Streams record and make one HTTP request to send it to Kinesis Streams\. With batching, each HTTP request can carry multiple records instead of just one\.

The KPL supports two types of batching:

+ *Aggregation* – Storing multiple records within a single Kinesis Streams record\. 

+ *Collection* – Using the API operation `PutRecords` to send multiple Kinesis Streams records to one or more shards in your Kinesis stream\. 

The two types of KPL batching are designed to co\-exist and can be turned on or off independently of one another\. By default, both are turned on\.

## Aggregation<a name="w3ab1c11b7b7c19c11"></a>

*Aggregation* refers to the storage of multiple records in a Kinesis Streams record\. Aggregation allows customers to increase the number of records sent per API call, which effectively increases producer throughput\.

Kinesis Streams shards support up to 1,000 Kinesis Streams records per second, or 1 MB throughput\. The Kinesis Streams records per second limit binds customers with records smaller than 1 KB\. Record aggregation allows customers to combine multiple records into a single Kinesis Streams record\. This allows customers to improve their per shard throughput\. 

Consider the case of one shard in region us\-east\-1 that is currently running at a constant rate of 1,000 records per second, with records that are 512 bytes each\. With KPL aggregation, you can pack one thousand records into only 10 Kinesis Streams records, reducing the RPS to 10 \(at 50 KB each\)\.

## Collection<a name="w3ab1c11b7b7c19c13"></a>

*Collection* refers to batching multiple Kinesis Streams records and sending them in a single HTTP request with a call to the API operation `PutRecords`, instead of sending each Kinesis Streams record in its own HTTP request\.

This increases throughput compared to using no collection because it reduces the overhead of making many separate HTTP requests\. In fact, `PutRecords` itself was specifically designed for this purpose\.

Collection differs from aggregation in that it is working with groups of Kinesis Streams records\. The Kinesis Streams records being collected can still contain multiple records from the user\. The relationship can be visualized as such:

```
record 0 --|
record 1   |        [ Aggregation ]
    ...    |--> Amazon Kinesis record 0 --|
    ...    |                              |
record A --|                              |
                                          |
    ...                   ...             |
                                          |
record K --|                              |
record L   |                              |      [ Collection ]
    ...    |--> Amazon Kinesis record C --|--> PutRecords Request
    ...    |                              |
record S --|                              |
                                          |
    ...                   ...             |
                                          |
record AA--|                              |
record BB  |                              |
    ...    |--> Amazon Kinesis record M --|
    ...    |
record ZZ--|
```