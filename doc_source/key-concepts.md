# Amazon Kinesis Data Streams Key Concepts<a name="key-concepts"></a>

As you get started with Amazon Kinesis Data Streams, you'll benefit from understanding its architecture and terminology\.

## Kinesis Data Streams High\-Level Architecture<a name="high-level-architecture"></a>

The following diagram illustrates the high\-level architecture of Kinesis Data Streams\. The producers continually push data to Kinesis Data Streams and the consumers process the data in real time\. Consumers \(such as a custom application running on Amazon EC2, or an Amazon Kinesis Data Firehose delivery stream\) can store their results using an AWS service such as Amazon DynamoDB, Amazon Redshift, or Amazon S3\. 

![\[Kinesis Data Streams High-level Architecture Diagram\]](http://docs.aws.amazon.com/streams/latest/dev/images/architecture.png)

## Kinesis Data Streams Terminology<a name="terminology"></a>

### Kinesis Data Streams<a name="stream"></a>

A *Kinesis data stream* is a set of [shards](#shard)\. Each shard has a sequence of data records\. Each data record has a [sequence number](#sequence-number) that is assigned by Kinesis Data Streams\. 

### Data Records<a name="data-record"></a>

A *data record* is the unit of data stored in a [Kinesis data stream](#stream)\. Data records are composed of a [sequence number](#sequence-number), [partition key](#partition-key), and data blob, which is an immutable sequence of bytes\. Kinesis Data Streams does not inspect, interpret, or change the data in the blob in any way\. A data blob can be up to 1 MB\.

### Retention Period<a name="w3ab1b5c15b7b6"></a>

The length of time data records are accessible after they are added to the stream\. A stream’s retention period is set to a default of 24 hours after creation\. You can increase the retention period up to 168 hours \(7 days\) using the [IncreaseStreamRetentionPeriod](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_IncreaseStreamRetentionPeriod.html) operation, and decrease the retention period down to a minimum of 24 hours using the [DecreaseStreamRetentionPeriod](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DecreaseStreamRetentionPeriod.html) operation\. Additional charges apply for streams with a retention period set to more than 24 hours\. For more information, see [Amazon Kinesis Data Streams Pricing](https://aws.amazon.com/kinesis/pricing/)\.

### Producers<a name="producers"></a>

Producers put records into Amazon Kinesis Data Streams\. For example, a web server sending log data to a stream is a producer\.

### Consumers<a name="consumers"></a>

Consumers get records from Amazon Kinesis Data Streams and process them\. These consumers are known as [Amazon Kinesis Data Streams Applications](#enabled-application)\.

### Amazon Kinesis Data Streams Applications<a name="enabled-application"></a>

An *Amazon Kinesis Data Streams application* is a consumer of a stream that commonly runs on a fleet of EC2 instances\.

You can develop an Amazon Kinesis Data Streams application using the Kinesis Client Library or using the Kinesis Data Streams API\.

The output of an Amazon Kinesis Data Streams application may be input for another stream, enabling you to create complex topologies that process data in real time\. An application can also send data to a variety of other AWS services\. There can be multiple applications for one stream, and each application can consume data from the stream independently and concurrently\.

### Shards<a name="shard"></a>

A *shard* is a uniquely identified sequence of data records in a stream\. A stream is composed of one or more shards, each of which provides a fixed unit of capacity\. Each shard can support up to 5 transactions per second for reads, up to a maximum total data read rate of 2 MB per second and up to 1,000 records per second for writes, up to a maximum total data write rate of 1 MB per second \(including partition keys\)\. The data capacity of your stream is a function of the number of shards that you specify for the stream\. The total capacity of the stream is the sum of the capacities of its shards\.

If your data rate increases, you can increase or decrease the number of shards allocated to your stream\.

### Partition Keys<a name="partition-key"></a>

A *partition key* is used to group data by shard within a stream\. The Kinesis Data Streams service segregates the data records belonging to a stream into multiple shards, using the partition key associated with each data record to determine which shard a given data record belongs to\. Partition keys are Unicode strings with a maximum length limit of 256 bytes\. An MD5 hash function is used to map partition keys to 128\-bit integer values and to map associated data records to shards\. When an application puts data into a stream, it must specfy a partition key\. 

### Sequence Numbers<a name="sequence-number"></a>

Each data record has a sequence number that is unique per partition-key within its shard\. The sequence number is assigned by Kinesis Data Streams after you write to the stream with `client.putRecords` or `client.putRecord`\. Sequence numbers for the same partition key generally increase over time; the longer the time period between write requests, the larger the sequence numbers become\.

**Note**  
Sequence numbers cannot be used as indexes to sets of data within the same stream\. To logically separate sets of data, use partition keys or create a separate stream for each dataset\.

### Kinesis Client Library<a name="client-library"></a>

The Kinesis Client Library is compiled into your application to enable fault\-tolerant consumption of data from the stream\. The Kinesis Client Library ensures that for every shard there is a record processor running and processing that shard\. The library also simplifies reading data from the stream\. The Kinesis Client Library uses an Amazon DynamoDB table to store control data\. It creates one table per application that is processing data\. 

### Application Name<a name="application-name"></a>

The name of an Amazon Kinesis Data Streams application identifies the application\. Each of your applications must have a unique name that is scoped to the AWS account and region used by the application\. This name is used as a name for the control table in Amazon DynamoDB and the namespace for Amazon CloudWatch metrics\. 

### Server\-side encryption<a name="server-side-encryption-concept"></a>

Amazon Kinesis Data Streams can automatically encrypt sensitive data as a producer enters it into a stream\. Kinesis Data Streams uses [KMS](http://docs.aws.amazon.com/kms/latest/developerguide/) master keys for encryption\. For more information, see [Using Server\-Side Encryption](server-side-encryption.md)\.

**Note**  
To read from or write to an encrypted stream, producer and consumer applications must have permission to access the master key\. For information on granting permissions to producer and consumer applications, see [Permissions to Use User\-Generated KMS Master Keys](permissions-user-key-KMS.md)\.

**Note**  
Using server\-side encryption incurs KMS costs\. For more information, see [AWS Key Management Service Pricing](http://aws.amazon.com/kms/pricing)\.
