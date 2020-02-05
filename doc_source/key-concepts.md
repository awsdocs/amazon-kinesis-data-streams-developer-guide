# Amazon Kinesis Data Streams Terminology and Concepts<a name="key-concepts"></a>

As you get started with Amazon Kinesis Data Streams, you can benefit from understanding its architecture and terminology\.

**Topics**
+ [Kinesis Data Streams High\-Level Architecture](#high-level-architecture)
+ [Kinesis Data Streams Terminology](#terminology)

## Kinesis Data Streams High\-Level Architecture<a name="high-level-architecture"></a>

The following diagram illustrates the high\-level architecture of Kinesis Data Streams\. The *producers* continually push data to Kinesis Data Streams, and the *consumers* process the data in real time\. Consumers \(such as a custom application running on Amazon EC2 or an Amazon Kinesis Data Firehose delivery stream\) can store their results using an AWS service such as Amazon DynamoDB, Amazon Redshift, or Amazon S3\. 

![\[Kinesis Data Streams high-level architecture diagram\]](http://docs.aws.amazon.com/streams/latest/dev/images/architecture.png)

## Kinesis Data Streams Terminology<a name="terminology"></a>

### Kinesis Data Stream<a name="stream"></a>

A *Kinesis data stream* is a set of [shards](#shard)\. Each shard has a sequence of data records\. Each data record has a [sequence number](#sequence-number) that is assigned by Kinesis Data Streams\. 

### Data Record<a name="data-record"></a>

A *data record* is the unit of data stored in a [Kinesis data stream](#stream)\. Data records are composed of a [sequence number](#sequence-number), a [partition key](#partition-key), and a data blob, which is an immutable sequence of bytes\. Kinesis Data Streams does not inspect, interpret, or change the data in the blob in any way\. A data blob can be up to 1 MB\.

### Retention Period<a name="retention"></a>

The *retention period* is the length of time that data records are accessible after they are added to the stream\. A stream’s retention period is set to a default of 24 hours after creation\. You can increase the retention period up to 168 hours \(7 days\) using the [IncreaseStreamRetentionPeriod](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_IncreaseStreamRetentionPeriod.html) operation, and decrease the retention period down to a minimum of 24 hours using the [DecreaseStreamRetentionPeriod](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_DecreaseStreamRetentionPeriod.html) operation\. Additional charges apply for streams with a retention period set to more than 24 hours\. For more information, see [Amazon Kinesis Data Streams Pricing](https://aws.amazon.com/kinesis/pricing/)\.

### Producer<a name="producers"></a>

*Producers* put records into Amazon Kinesis Data Streams\. For example, a web server sending log data to a stream is a producer\.

### Consumer<a name="consumers"></a>

*Consumers* get records from Amazon Kinesis Data Streams and process them\. These consumers are known as [Amazon Kinesis Data Streams Application](#enabled-application)\.

### Amazon Kinesis Data Streams Application<a name="enabled-application"></a>

An *Amazon Kinesis Data Streams application* is a consumer of a stream that commonly runs on a fleet of EC2 instances\.

There are two types of consumers that you can develop: shared fan\-out consumers and enhanced fan\-out consumers\. To learn about the differences between them, and to see how you can create each type of consumer, see [Reading Data from Amazon Kinesis Data Streams](building-consumers.md)\.

The output of a Kinesis Data Streams application can be input for another stream, enabling you to create complex topologies that process data in real time\. An application can also send data to a variety of other AWS services\. There can be multiple applications for one stream, and each application can consume data from the stream independently and concurrently\.

### Shard<a name="shard"></a>

A *shard* is a uniquely identified sequence of data records in a stream\. A stream is composed of one or more shards, each of which provides a fixed unit of capacity\. Each shard can support up to 5 transactions per second for reads, up to a maximum total data read rate of 2 MB per second and up to 1,000 records per second for writes, up to a maximum total data write rate of 1 MB per second \(including partition keys\)\. The data capacity of your stream is a function of the number of shards that you specify for the stream\. The total capacity of the stream is the sum of the capacities of its shards\.

If your data rate increases, you can increase or decrease the number of shards allocated to your stream\. For more information, see [Resharding a Stream](kinesis-using-sdk-java-resharding.md)\.

### Partition Key<a name="partition-key"></a>

A *partition key* is used to group data by shard within a stream\. Kinesis Data Streams segregates the data records belonging to a stream into multiple shards\. It uses the partition key that is associated with each data record to determine which shard a given data record belongs to\. Partition keys are Unicode strings, with a maximum length limit of 256 characters for each key\. An MD5 hash function is used to map partition keys to 128\-bit integer values and to map associated data records to shards using the hash key ranges of the shards\. When an application puts data into a stream, it must specify a partition key\. 

### Sequence Number<a name="sequence-number"></a>

Each data record has a *sequence number* that is unique per partition\-key within its shard\. Kinesis Data Streams assigns the sequence number after you write to the stream with `client.putRecords` or `client.putRecord`\. Sequence numbers for the same partition key generally increase over time\. The longer the time period between write requests, the larger the sequence numbers become\.

**Note**  
Sequence numbers cannot be used as indexes to sets of data within the same stream\. To logically separate sets of data, use partition keys or create a separate stream for each dataset\.

### Kinesis Client Library<a name="client-library"></a>

The Kinesis Client Library is compiled into your application to enable fault\-tolerant consumption of data from the stream\. The Kinesis Client Library ensures that for every shard there is a record processor running and processing that shard\. The library also simplifies reading data from the stream\. The Kinesis Client Library uses an Amazon DynamoDB table to store control data\. It creates one table per application that is processing data\.

There are two major versions of the Kinesis Client Library\. Which one you use depends on the type of consumer you want to create\. For more information, see [Reading Data from Amazon Kinesis Data Streams](building-consumers.md)\. 

### Application Name<a name="application-name"></a>

The name of an Amazon Kinesis Data Streams application identifies the application\. Each of your applications must have a unique name that is scoped to the AWS account and Region used by the application\. This name is used as a name for the control table in Amazon DynamoDB and the namespace for Amazon CloudWatch metrics\. 

### Server\-Side Encryption<a name="server-side-encryption-concept"></a>

Amazon Kinesis Data Streams can automatically encrypt sensitive data as a producer enters it into a stream\. Kinesis Data Streams uses [AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/) master keys for encryption\. For more information, see [Data Protection in Amazon Kinesis Data Streams](server-side-encryption.md)\.

**Note**  
To read from or write to an encrypted stream, producer and consumer applications must have permission to access the master key\. For information about granting permissions to producer and consumer applications, see [Permissions to Use User\-Generated KMS Master Keys](permissions-user-key-KMS.md)\.

**Note**  
Using server\-side encryption incurs AWS Key Management Service \(AWS KMS\) costs\. For more information, see [AWS Key Management Service Pricing](http://aws.amazon.com/kms/pricing)\.