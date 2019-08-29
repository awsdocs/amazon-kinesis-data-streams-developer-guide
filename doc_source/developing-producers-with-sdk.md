# Developing Producers Using the Amazon Kinesis Data Streams API with the AWS SDK for Java<a name="developing-producers-with-sdk"></a>

You can develop producers using the Amazon Kinesis Data Streams API with the AWS SDK for Java\. If you are new to Kinesis Data Streams, start by becoming familiar with the concepts and terminology presented in [What Is Amazon Kinesis Data Streams?](introduction.md) and [Getting Started with Amazon Kinesis Data Streams](getting-started.md)\.

These examples discuss the [Kinesis Data Streams API](https://docs.aws.amazon.com/kinesis/latest/APIReference/) and use the [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) to add \(put\) data to a stream\. However, for most use cases, you should prefer the Kinesis Data Streams KPL library\. For more information, see [Developing Producers Using the Amazon Kinesis Producer Library](developing-producers-with-kpl.md)\.

The Java example code in this chapter demonstrates how to perform basic Kinesis Data Streams API operations, and is divided up logically by operation type\. These examples do not represent production\-ready code, in that they do not check for all possible exceptions, or account for all possible security or performance considerations\. Also, you can call the [Kinesis Data Streams API](https://docs.aws.amazon.com/kinesis/latest/APIReference/) using other programming languages\. For more information about all available AWS SDKs, see [Start Developing with Amazon Web Services](https://aws.amazon.com/developers/getting-started/)\.

Each task has prerequisites; for example, you cannot add data to a stream until you have created a stream, which requires you to create a client \. For more information, see [Creating and Managing Streams](working-with-streams.md)\.

## Adding Data to a Stream<a name="kinesis-using-sdk-java-add-data-to-stream"></a>

Once a stream is created, you can add data to it in the form of records\. A record is a data structure that contains the data to be processed in the form of a data blob\. After you store the data in the record, Kinesis Data Streams does not inspect, interpret, or change the data in any way\. Each record also has an associated sequence number and partition key\.

There are two different operations in the Kinesis Data Streams API that add data to a stream, [https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecords.html](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecords.html) and [https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html)\. The `PutRecords` operation sends multiple records to your stream per HTTP request, and the singular `PutRecord` operation sends records to your stream one at a time \(a separate HTTP request is required for each record\)\. You should prefer using `PutRecords` for most applications because it will achieve higher throughput per data producer\. For more information about each of these operations, see the separate subsections below\.

**Topics**
+ [Adding Multiple Records with PutRecords](#kinesis-using-sdk-java-putrecords)
+ [Adding a Single Record with PutRecord](#kinesis-using-sdk-java-putrecord)

Always keep in mind that, as your source application is adding data to the stream using the Kinesis Data Streams API, there are most likely one or more consumer applications that are simultaneously processing data off the stream\. For information about how consumers get data using the Kinesis Data Streams API, see [Getting Data from a Stream](developing-consumers-with-sdk.md#kinesis-using-sdk-java-get-data)\.

**Important**  
[Changing the Data Retention Period](kinesis-extended-retention.md)

### Adding Multiple Records with PutRecords<a name="kinesis-using-sdk-java-putrecords"></a>

The [https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecords.html](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecords.html) operation sends multiple records to Kinesis Data Streams in a single request\. By using `PutRecords`, producers can achieve higher throughput when sending data to their Kinesis data stream\. Each `PutRecords` request can support up to 500 records\. Each record in the request can be as large as 1 MB, up to a limit of 5 MB for the entire request, including partition keys\. As with the single `PutRecord` operation described below, `PutRecords` uses sequence numbers and partition keys\. However, the `PutRecord` parameter `SequenceNumberForOrdering` is not included in a `PutRecords` call\. The `PutRecords` operation attempts to process all records in the natural order of the request\. 

Each data record has a unique sequence number\. The sequence number is assigned by Kinesis Data Streams after you call `client.putRecords` to add the data records to the stream\. Sequence numbers for the same partition key generally increase over time; the longer the time period between `PutRecords` requests, the larger the sequence numbers become\.

**Note**  
Sequence numbers cannot be used as indexes to sets of data within the same stream\. To logically separate sets of data, use partition keys or create a separate stream for each data set\.

A `PutRecords` request can include records with different partition keys\. The scope of the request is a stream; each request may include any combination of partition keys and records up to the request limits\. Requests made with many different partition keys to streams with many different shards are generally faster than requests with a small number of partition keys to a small number of shards\. The number of partition keys should be much larger than the number of shards to reduce latency and maximize throughput\.

#### PutRecords Example<a name="kinesis-using-sdk-java-putrecords-example"></a>

The following code creates 100 data records with sequential partition keys and puts them in a stream called `DataStream`\. 

```
        AmazonKinesisClientBuilder clientBuilder = AmazonKinesisClientBuilder.standard();
        
        clientBuilder.setRegion(regionName);
        clientBuilder.setCredentials(credentialsProvider);
        clientBuilder.setClientConfiguration(config);
        
        AmazonKinesis kinesisClient = clientBuilder.build();
 
        PutRecordsRequest putRecordsRequest  = new PutRecordsRequest();
        putRecordsRequest.setStreamName(streamName);
        List <PutRecordsRequestEntry> putRecordsRequestEntryList  = new ArrayList<>(); 
        for (int i = 0; i < 100; i++) {
            PutRecordsRequestEntry putRecordsRequestEntry  = new PutRecordsRequestEntry();
            putRecordsRequestEntry.setData(ByteBuffer.wrap(String.valueOf(i).getBytes()));
            putRecordsRequestEntry.setPartitionKey(String.format("partitionKey-%d", i));
            putRecordsRequestEntryList.add(putRecordsRequestEntry); 
        }

        putRecordsRequest.setRecords(putRecordsRequestEntryList);
        PutRecordsResult putRecordsResult  = kinesisClient.putRecords(putRecordsRequest);
        System.out.println("Put Result" + putRecordsResult);
```

The `PutRecords` response includes an array of response `Records`\. Each record in the response array directly correlates with a record in the request array using natural ordering, from the top to the bottom of the request and response\. The response `Records` array always includes the same number of records as the request array\.

#### Handling Failures When Using PutRecords<a name="kinesis-using-sdk-java-putrecords-handling-failures"></a>

By default, failure of individual records within a request does not stop the processing of subsequent records in a `PutRecords` request\. This means that a response `Records` array includes both successfully and unsuccessfully processed records\. You must detect unsuccessfully processed records and include them in a subsequent call\. 

Successful records include `SequenceNumber` and `ShardID` values, and unsuccessful records include `ErrorCode` and `ErrorMessage` values\. The `ErrorCode` parameter reflects the type of error and can be one of the following values: `ProvisionedThroughputExceededException` or `InternalFailure`\. `ErrorMessage` provides more detailed information about the `ProvisionedThroughputExceededException` exception including the account ID, stream name, and shard ID of the record that was throttled\. The example below has three records in a `PutRecords` request\. The second record fails and is reflected in the response\. 

**Example PutRecords Request Syntax**  

```
{
    "Records": [
        {
    	"Data": "XzxkYXRhPl8w",
	    "PartitionKey": "partitionKey1"
        },
        {
    	"Data": "AbceddeRFfg12asd",
	    "PartitionKey": "partitionKey1"	
        },
        {
    	"Data": "KFpcd98*7nd1",
	    "PartitionKey": "partitionKey3"
        }
    ],
    "StreamName": "myStream"
}
```

**Example PutRecords Response Syntax**  

```
{
    "FailedRecordCount”: 1,
    "Records": [
        {
	    "SequenceNumber": "21269319989900637946712965403778482371",
	    "ShardId": "shardId-000000000001"

        },
        {
	    “ErrorCode":”ProvisionedThroughputExceededException”,
	    “ErrorMessage": "Rate exceeded for shard shardId-000000000001 in stream exampleStreamName under account 111111111111."

        },
        {
	    "SequenceNumber": "21269319989999637946712965403778482985",
	    "ShardId": "shardId-000000000002"
        }
    ]
}
```

Records that were unsuccessfully processed can be included in subsequent `PutRecords` requests\. First, check the `FailedRecordCount` parameter in the `putRecordsResult` to confirm if there are failed records in the request\. If so, each `putRecordsEntry` that has an `ErrorCode` that is not `null` should be added to a subsequent request\. For an example of this type of handler, refer to the following code\.

**Example PutRecords failure handler**  

```
PutRecordsRequest putRecordsRequest = new PutRecordsRequest();
putRecordsRequest.setStreamName(myStreamName);
List<PutRecordsRequestEntry> putRecordsRequestEntryList = new ArrayList<>();
for (int j = 0; j < 100; j++) {
    PutRecordsRequestEntry putRecordsRequestEntry = new PutRecordsRequestEntry();
    putRecordsRequestEntry.setData(ByteBuffer.wrap(String.valueOf(j).getBytes()));
    putRecordsRequestEntry.setPartitionKey(String.format("partitionKey-%d", j));
    putRecordsRequestEntryList.add(putRecordsRequestEntry);
}

putRecordsRequest.setRecords(putRecordsRequestEntryList);
PutRecordsResult putRecordsResult = amazonKinesisClient.putRecords(putRecordsRequest);

while (putRecordsResult.getFailedRecordCount() > 0) {
    final List<PutRecordsRequestEntry> failedRecordsList = new ArrayList<>();
    final List<PutRecordsResultEntry> putRecordsResultEntryList = putRecordsResult.getRecords();
    for (int i = 0; i < putRecordsResultEntryList.size(); i++) {
        final PutRecordsRequestEntry putRecordRequestEntry = putRecordsRequestEntryList.get(i);
        final PutRecordsResultEntry putRecordsResultEntry = putRecordsResultEntryList.get(i);
        if (putRecordsResultEntry.getErrorCode() != null) {
            failedRecordsList.add(putRecordRequestEntry);
        }
    }
    putRecordsRequestEntryList = failedRecordsList;
    putRecordsRequest.setRecords(putRecordsRequestEntryList);
    putRecordsResult = amazonKinesisClient.putRecords(putRecordsRequest);
}
```

### Adding a Single Record with PutRecord<a name="kinesis-using-sdk-java-putrecord"></a>

Each call to [https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html) operates on a single record\. Prefer the `PutRecords` operation described in [Adding Multiple Records with PutRecords](#kinesis-using-sdk-java-putrecords) unless your application specifically needs to always send single records per request, or some other reason `PutRecords` can't be used\.

Each data record has a unique sequence number\. The sequence number is assigned by Kinesis Data Streams after you call `client.putRecord` to add the data record to the stream\. Sequence numbers for the same partition key generally increase over time; the longer the time period between `PutRecord` requests, the larger the sequence numbers become\.

 When puts occur in quick succession, the returned sequence numbers are not guaranteed to increase because the put operations appear essentially as simultaneous to Kinesis Data Streams\. To guarantee strictly increasing sequence numbers for the same partition key, use the `SequenceNumberForOrdering` parameter, as shown in the [PutRecord Example](#kinesis-using-sdk-java-putrecord-example) code sample\. 

 Whether or not you use `SequenceNumberForOrdering`, records that Kinesis Data Streams receives through a `GetRecords` call are strictly ordered by sequence number\. 

**Note**  
Sequence numbers cannot be used as indexes to sets of data within the same stream\. To logically separate sets of data, use partition keys or create a separate stream for each data set\.

A partition key is used to group data within the stream\. A data record is assigned to a shard within the stream based on its partition key\. Specifically, Kinesis Data Streams uses the partition key as input to a hash function that maps the partition key \(and associated data\) to a specific shard\.

 As a result of this hashing mechanism, all data records with the same partition key map to the same shard within the stream\. However, if the number of partition keys exceeds the number of shards, some shards necessarily contain records with different partition keys\. From a design standpoint, to ensure that all your shards are well utilized, the number of shards \(specified by the `setShardCount` method of `CreateStreamRequest`\) should be substantially less than the number of unique partition keys, and the amount of data flowing to a single partition key should be substantially less than the capacity of the shard\. 

#### PutRecord Example<a name="kinesis-using-sdk-java-putrecord-example"></a>

The following code creates ten data records, distributed across two partition keys, and puts them in a stream called `myStreamName`\.

```
for (int j = 0; j < 10; j++) 
{
  PutRecordRequest putRecordRequest = new PutRecordRequest();
  putRecordRequest.setStreamName( myStreamName );
  putRecordRequest.setData(ByteBuffer.wrap( String.format( "testData-%d", j ).getBytes() ));
  putRecordRequest.setPartitionKey( String.format( "partitionKey-%d", j/5 ));  
  putRecordRequest.setSequenceNumberForOrdering( sequenceNumberOfPreviousRecord );
  PutRecordResult putRecordResult = client.putRecord( putRecordRequest );
  sequenceNumberOfPreviousRecord = putRecordResult.getSequenceNumber();
}
```

The preceding code sample uses `setSequenceNumberForOrdering` to guarantee strictly increasing ordering within each partition key\. To use this parameter effectively, set the `SequenceNumberForOrdering` of the current record \(record *n*\) to the sequence number of the preceding record \(record *n\-1*\)\. To get the sequence number of a record that has been added to the stream, call `getSequenceNumber` on the result of `putRecord`\.

The `SequenceNumberForOrdering` parameter ensures strictly increasing sequence numbers for the same partition key\. `SequenceNumberForOrdering` does not provide ordering of records across multiple partition keys\. 