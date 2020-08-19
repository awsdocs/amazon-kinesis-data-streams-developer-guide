# Using the Kinesis Client Library<a name="shared-throughput-kcl-consumers"></a>

One of the methods of developing custom consumer applications that can process data from KDS data streams is to use the Kinesis Client Library \(KCL\)\.

**Topics**
+ [What is the Kinesis Client Library?](#shared-throughput-kcl-consumers-overview)
+ [KCL Available Versions](#shared-throughput-kcl-consumers-versions)
+ [KCL Concepts](#shared-throughput-kcl-consumers-concepts)
+ [Using a Lease Table to Track the Shards Processed by the KCL Consumer Application](#shared-throughput-kcl-consumers-leasetable)

## What is the Kinesis Client Library?<a name="shared-throughput-kcl-consumers-overview"></a>

KCL helps you consume and process data from a Kinesis data stream by taking care of many of the complex tasks associated with distributed computing\. These include load balancing across multiple consumer application instances, responding to consumer application instance failures, checkpointing processed records, and reacting to resharding\. The KCL takes care of all of these subtasks so that you can focus your efforts on writing your custom record\-processing logic\.

The KCL is different from the Kinesis Data Streams APIs that are available in the AWS SDKs\. The Kinesis Data Streams APIs help you manage many aspects of Kinesis Data Streams, including creating streams, resharding, and putting and getting records\. The KCL provides a layer of abstraction around all these subtasks, specifically so that you can focus on your consumer application’s custom data processing logic\. For information about the Kinesis Data Streams API, see the [Amazon Kinesis API Reference](https://docs.aws.amazon.com/kinesis/latest/APIReference/Welcome.html)\.

**Important**  
The KCL is a Java library\. Support for languages other than Java is provided using a multi\-language interface called the MultiLangDaemon\. This daemon is Java\-based and runs in the background when you are using a KCL language other than Java\. For example, if you install the KCL for Python and write your consumer application entirely in Python, you still need Java installed on your system because of the MultiLangDaemon\. Further, MultiLangDaemon has some default settings that you might need to customize for your use case, for example, the AWS region that it connects to\. For more information about the MultiLangDaemon on GitHub, see [KCL MultiLangDaemon project](https://github.com/awslabs/amazon-kinesis-client/tree/v1.x/src/main/java/com/amazonaws/services/kinesis/multilang)\.

The KCL acts as an intermediary between your record processing logic and Kinesis Data Streams\. The KCL performs the following tasks:
+ Connects to the data stream 
+ Enumerates the shards within the data stream
+ Uses leases to coordinates shard associations with its workers 
+ Instantiates a record processor for every shard it manages 
+ Pulls data records from the data stream 
+ Pushes the records to the corresponding record processor 
+ Checkpoints processed records 
+ Balances shard\-worker associations \(leases\) when the worker instance count changes or when the data stream is resharded \(shards are split or merged\)

## KCL Available Versions<a name="shared-throughput-kcl-consumers-versions"></a>

Currently, you can use either of the following supported versions of KCL to build your custom consumer applications:
+ **KCL 1\.x**

  For more information, see [Developing KCL 1\.x Consumers](developing-consumers-with-kcl.md)
+ **KCL 2\.x**

  For more information, see [Developing KCL 2\.x Consumers](developing-consumers-with-kcl-v2.md)

You can use either KCL 1\.x or KCL 2\.x to build consumer applications that use shared throughput\. For more information, see [Developing Custom Consumers with Shared Throughput Using KCL](custom-kcl-consumers.md)\.

To build consumer applications that use dedicated throughput \(enhanced fan\-out consumers\), you can only use KCL 2\.x\. For more information, see [Developing Custom Consumers with Dedicated Throughput \(Enhanced Fan\-Out\)](enhanced-consumers.md)\.

For information about the differences between KCL 1\.x and KCL 2\.x, and instructions on how to migrate from KCL 1\.x to KCL 2\.x, see [Migrating Consumers from KCL 1\.x to KCL 2\.x](kcl-migration.md)\.

## KCL Concepts<a name="shared-throughput-kcl-consumers-concepts"></a>
+ **KCL consumer application** – an application that is custom\-built using KCL and designed to read and process records from data streams\. 
+ **Consumer application instance** \- KCL consumer applications are typically distributed, with one or more application instances running simultaneously in order to coordinate on failures and dynamically load balance data record processing\.
+ **Worker** – a high level class that a KCL consumer application instance uses to start processing data\. 
**Important**  
Each KCL consumer application instance has one worker\. 

  The worker initializes and oversees various tasks, including syncing shard and lease information, tracking shard assignments, and processing data from the shards\. A worker provides KCL with the configuration information for the consumer application, such as the name of the data stream whose data records this KCL consumer application is going to process and the AWS credentials that are needed to access this data stream\. The worker also kick starts that specific KCL consumer application instance to deliver data records from the data stream to the record processors\.
**Important**  
In KCL 1\.x this class is called **Worker**\. For more information, \(these are the Java KCL repositories\), see [https://github\.com/awslabs/amazon\-kinesis\-client/blob/v1\.x/src/main/java/com/amazonaws/services/kinesis/clientlibrary/lib/worker/Worker\.java](https://github.com/awslabs/amazon-kinesis-client/blob/v1.x/src/main/java/com/amazonaws/services/kinesis/clientlibrary/lib/worker/Worker.java)\. In KCL 2\.x, this class is called **Scheduler**\. Scheduler’s purpose in KCL 2\.x is identical to Worker’s purpose in KCL 1\.x\. For more information about the Scheduler class in KCL 2\.x, see [https://github\.com/awslabs/amazon\-kinesis\-client/blob/master/amazon\-kinesis\-client/src/main/java/software/amazon/kinesis/coordinator/Scheduler\.java](https://github.com/awslabs/amazon-kinesis-client/blob/master/amazon-kinesis-client/src/main/java/software/amazon/kinesis/coordinator/Scheduler.java)\. 
+ **Lease** – data that defines the binding between a worker and a shard\. Distributed KCL consumer applications use leases to partition data record processing across a fleet of workers\. At any given time, each shard of data records is bound to a particular worker by a lease identified by the **leaseKey** variable\. 

  By default, a worker can hold one or more leases \(subject to the value of the **maxLeasesForWorker** variable\) at the same time\. 
**Important**  
Every worker will contend to hold all available leases for all available shards in a data stream\. But only one worker will successfully hold each lease at any one time\. 

  For example, if you have a consumer application instance A with worker A that is processing a data stream with 4 shards, worker A can hold leases to shards 1, 2, 3, and 4 at the same time\. But if you have two consumer application instances: A and B with worker A and worker B, and these instances are processing a data stream with 4 shards, worker A and worker B cannot both hold the lease to shard 1 at the same time\. One worker holds the lease to a particular shard until it is ready to stop processing this shard’s data records or until it fails\. When one worker stops holding the lease, another worker takes up and holds the lease\. 

  For more information, \(these are the Java KCL repositories\), see [https://github\.com/awslabs/amazon\-kinesis\-client/blob/v1\.x/src/main/java/com/amazonaws/services/kinesis/leases/impl/Lease\.java](https://github.com/awslabs/amazon-kinesis-client/blob/v1.x/src/main/java/com/amazonaws/services/kinesis/leases/impl/Lease.java) for KCL 1\.x and [https://github\.com/awslabs/amazon\-kinesis\-client/blob/master/amazon\-kinesis\-client/src/main/java/software/amazon/kinesis/leases/Lease\.java](https://github.com/awslabs/amazon-kinesis-client/blob/master/amazon-kinesis-client/src/main/java/software/amazon/kinesis/leases/Lease.java) for KCL 2\.x\.
+ **Lease table** \- a unique Amazon DynamoDB table that is used to keep track of the shards in a KDS data stream that are being leased and processed by the workers of the KCL consumer application\. The lease table must remain in sync \(within a worker and across all workers\) with the latest shard information from the data stream while the KCL consumer application is running\. For more information, see [Using a Lease Table to Track the Shards Processed by the KCL Consumer Application](#shared-throughput-kcl-consumers-leasetable)\.
+ **Record processor** – the logic that defines how your KCL consumer application processes the data that it gets from the data streams\. At runtime, a KCL consumer application instance instantiates a worker, and this worker instantiates one record processor for every shard to which it holds a lease\. 

## Using a Lease Table to Track the Shards Processed by the KCL Consumer Application<a name="shared-throughput-kcl-consumers-leasetable"></a>

**Topics**
+ [What Is a Lease Table](#shared-throughput-kcl-consumers-what-is-leasetable)
+ [Throughput](#shared-throughput-kcl-leasetable-throughput)
+ [How a Lease Table Is Synchronized with the Shards in a KDS Data Stream](#shared-throughput-kcl-consumers-leasetable-sync)

### What Is a Lease Table<a name="shared-throughput-kcl-consumers-what-is-leasetable"></a>

For each Amazon Kinesis Data Streams application, KCL uses a unique lease table \(stored in a Amazon DynamoDB table\) to keep track of the shards in a KDS data stream that are being leased and processed by the workers of the KCL consumer application\.

**Important**  
KCL uses the name of the consumer application to create the name of the lease table that this consumer application uses, therefore each consumer application name must be unique\.

You can view the lease table using the [Amazon DynamoDB console](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ConsoleDynamoDB.html) while the consumer application is running\.

If the lease table for your KCL consumer application does not exist when the application starts up, one of the workers creates the lease table for this application\. 

**Important**  
 Your account is charged for the costs associated with the DynamoDB table, in addition to the costs associated with Kinesis Data Streams itself\. 

Each row in the lease table represents a shard that is being processed by the workers of your consumer application\. The hash key for the lease table is **leaseKey**, which is the shard ID\.

In addition to the shard ID, each row also includes the following data:
+ **checkpoint:** The most recent checkpoint sequence number for the shard\. This value is unique across all shards in the data stream\.
+ **checkpointSubSequenceNumber:** When using the Kinesis Producer Library's aggregation feature, this is an extension to **checkpoint** that tracks individual user records within the Kinesis record\.
+ **leaseCounter:** Used for lease versioning so that workers can detect that their lease has been taken by another worker\.
+ **leaseKey:** A unique identifier for a lease\. Each lease is particular to a shard in the data stream and is held by one worker at a time\.
+ **leaseOwner:** The worker that is holding this lease\.
+ **ownerSwitchesSinceCheckpoint:** How many times this lease has changed workers since the last time a checkpoint was written\.
+ **parentShardId:** Used to ensure that the parent shard is fully processed before processing starts on the child shards\. This ensures that records are processed in the same order they were put into the stream\.
+ **hashrange:** Used by the `PeriodicShardSyncManager` to run periodic syncs to find missing shards in the lease table and create leases for them if required\. 
**Note**  
This data is present in the lease table for every shard starting with KCL 1\.14 and KCL 2\.3\. For more information about `PeriodicShardSyncManager` and periodic synchronization between leases and shards, see [How a Lease Table Is Synchronized with the Shards in a KDS Data Stream](#shared-throughput-kcl-consumers-leasetable-sync)\.
+ **childshards:** Used by the `LeaseCleanupManager` to review the child shard's processing status and decide whether the parent shard can be deleted from the lease table\.
**Note**  
This data is present in the lease table for every shard starting with KCL 1\.14 and KCL 2\.3\.

### Throughput<a name="shared-throughput-kcl-leasetable-throughput"></a>

If your Amazon Kinesis Data Streams application receives provisioned\-throughput exceptions, you should increase the provisioned throughput for the DynamoDB table\. The KCL creates the table with a provisioned throughput of 10 reads per second and 10 writes per second, but this might not be sufficient for your application\. For example, if your Amazon Kinesis Data Streams application does frequent checkpointing or operates on a stream that is composed of many shards, you might need more throughput\.

For information about provisioned throughput in DynamoDB, see [Read/Write Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html) and [Working with Tables and Data](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithDDTables.html) in the *Amazon DynamoDB Developer Guide*\.

### How a Lease Table Is Synchronized with the Shards in a KDS Data Stream<a name="shared-throughput-kcl-consumers-leasetable-sync"></a>

Workers in KCL consumer applications use leases to process shards from a given data stream\. The information on what worker is leasing what shard at any given time is stored in a lease table\. The lease table must remain in sync with the latest shard information from the data stream while the KCL consumer application is running\. KCL synchronizes the lease table with the shards information acquired from the Kinesis Data Streams service during the consumer application bootstraping \(either when the consumer application is initialized or restarted\) and also whenever a shard that is being processed reaches an end \(resharding\)\. In other words, the workers or a KCL consumer application are synchronized with the data stream that they are processing during the initial consumer application bootstrap and whenever the consumer application encounters a data stream reshard event\.

**Topics**
+ [Synchronization in KCL 1\.0 \- 1\.13 and KCL 2\.0 \- 2\.2](#shared-throughput-kcl-consumers-leasetable-sync-old)
+ [Synchronization in KCL 2\.x, Starting with KCL 2\.3 and Beyond](#shared-throughput-kcl-consumers-leasetable-sync-new-kcl2)
+ [Synchronization in KCL 1\.x, Starting with KCL 1\.14 and Beyond](#shared-throughput-kcl-consumers-leasetable-sync-new-kcl1)

#### Synchronization in KCL 1\.0 \- 1\.13 and KCL 2\.0 \- 2\.2<a name="shared-throughput-kcl-consumers-leasetable-sync-old"></a>

In KCL 1\.0 \- 1\.13 and KCL 2\.0 \- 2\.2, during consumer application's bootstraping and also during each data stream reshard event, KCL synchronizes the lease table with the shards information acquired from the Kinesis Data Streams service by invoking the `ListShards` or the `DescribeStream` discovery APIs\. In all the KCL versions listed above, each worker of a KCL consumer application completes the following steps to perform the lease/shard synchronization process during the consumer application's bootstrapping and at each stream reshard event:
+ Fetches all the shards for data the stream that is being processed
+ Fetches all the shard leases from the lease table
+ Filters out each open shard that does not have a lease in the lease table
+ Iterates over all found open shards and for each open shard with no open parent:
  + Traverses the hierarchy tree through its ancestors path to determine if the shard is a descendant\. A shard is considered a descendant, if an ancestor shard is being processed \(lease entry for ancestor shard exists in the lease table\) or if an ancestor shard should be processed \(for example, if the initial position is `TRIM_HORIZON` or `AT_TIMESTAMP`\)
  + If the open shard in context is a descendant, KCL checkpoints the shard based on initial position and creates leases for its parents, if required

#### Synchronization in KCL 2\.x, Starting with KCL 2\.3 and Beyond<a name="shared-throughput-kcl-consumers-leasetable-sync-new-kcl2"></a>

Starting with the latest supported versions of KCL 2\.x \(KCL 2\.3\) and beyond, the library now supports the following changes to the synchronization process\. These lease/shard synchronization changes significantly reduce the number of API calls made by KCL consumer applications to the Kinesis Data Streams service and optimize the lease management in your KCL consumer application\. 
+ During application's bootstraping, if the lease table is empty, KCL utilizes the `ListShard` API's filtering option \(the `ShardFilter` optional request parameter\) to retrieve and create leases only for a snapshot of shards open at the time specified by the `ShardFilter` parameter\. The `ShardFilter` parameter enables you to filter out the response of the `ListShards` API\. The only required property of the `ShardFilter` parameter is `Type`\. KCL uses the `Type` filter property and the following of its valid values to identify and return a snapshot of open shards that might require new leases:
  + `AT_TRIM_HORIZON` \- the response includes all the shards that were open at `TRIM_HORIZON`\. 
  + `AT_LATEST` \- the response includes only the currently open shards of the data stream\. 
  + `AT_TIMESTAMP` \- the response includes all shards whose start timestamp is less than or equal to the given timestamp and end timestamp is greater than or equal to the given timestamp or still open\.

  `ShardFilter` is used when creating leases for an empty lease table to initialize leases for a snapshot of shards specified at `RetrievalConfig#initialPositionInStreamExtended`\.

  For more information about `ShardFilter`, see [https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ShardFilter.html](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ShardFilter.html)\.
+ Instead of all workers performing the lease/shard synchronization to keep the lease table up to date with the latest shards in the data stream, a single elected worker leader performs the lease/shard synchronization\.
+ KCL 2\.3 uses the `ChildShards` return parameter of the `GetRecords` and the `SubscribeToShard` APIs to perform lease/shard synchronization that happens at `SHARD_END` for closed shards, allowing a KCL worker to only create leases for the child shards of the shard it finished processing\. For shared throughout consumer applications, this optimization of the lease/shard synchronization uses the `ChildShards` parameter of the `GetRecords` API\. For the dedicated throughput \(enhanced fan\-out\) consumer applications, this optimization of the lease/shard synchronization uses the `ChildShards` parameter of the `SubscribeToShard` API\. For more information, see [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html), [SubscribeToShards](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_SubscribeToShard.html), and [ChildShard](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ChildShard.html)\.
+ With the above changes, the behavior of KCL is moving from the model of all workers learning about all existing shards to the model of workers learning only about the children shards of the shards that each worker owns\. Therefore, in addition to the synchronization that happens during consumer application bootstraping and reshard events, KCL now also performs additional periodic shard/lease scans in order to identify any potential holes in the lease table \(in other words, to learn about all new shards\) to ensure the complete hash range of the data stream is being processed and create leases for them if required\. `PeriodicShardSyncManager` is the component that is responsible for running periodic lease/shard scans\. 

  For more information about `PeriodicShardSyncManager` in KCL 2\.3, see [https://github\.com/awslabs/amazon\-kinesis\-client/blob/master/amazon\-kinesis\-client/src/main/java/software/amazon/kinesis/leases/LeaseManagementConfig\.java\#L201\-L213](https://github.com/awslabs/amazon-kinesis-client/blob/master/amazon-kinesis-client/src/main/java/software/amazon/kinesis/leases/LeaseManagementConfig.java#L201-L213)\.

  In KCL 2\.3, new configuration options are available to configure `PeriodicShardSyncManager` in `LeaseManagementConfig`:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/streams/latest/dev/shared-throughput-kcl-consumers.html)

  New CloudWatch metrics are also now emitted to monitor the health of the `PeriodicShardSyncManager`\. For more information, see [PeriodicShardSyncManager](monitoring-with-kcl.md#periodic-task)\.
+ Including an optimization to `HierarchicalShardSyncer` to only create leases for one layer of shards\.

#### Synchronization in KCL 1\.x, Starting with KCL 1\.14 and Beyond<a name="shared-throughput-kcl-consumers-leasetable-sync-new-kcl1"></a>

Starting with the latest supported versions of KCL 1\.x \(KCL 1\.14\) and beyond, the library now supports the following changes to the synchronization process\. These lease/shard synchronization changes significantly reduce the number of API calls made by KCL consumer applications to the Kinesis Data Streams service and optimize the lease management in your KCL consumer application\. 
+ During application's bootstraping, if the lease table is empty, KCL utilizes the `ListShard` API's filtering option \(the `ShardFilter` optional request parameter\) to retrieve and create leases only for a snapshot of shards open at the time specified by the `ShardFilter` parameter\. The `ShardFilter` parameter enables you to filter out the response of the `ListShards` API\. The only required property of the `ShardFilter` parameter is `Type`\. KCL uses the `Type` filter property and the following of its valid values to identify and return a snapshot of open shards that might require new leases:
  + `AT_TRIM_HORIZON` \- the response includes all the shards that were open at `TRIM_HORIZON`\. 
  + `AT_LATEST` \- the response includes only the currently open shards of the data stream\. 
  + `AT_TIMESTAMP` \- the response includes all shards whose start timestamp is less than or equal to the given timestamp and end timestamp is greater than or equal to the given timestamp or still open\.

  `ShardFilter` is used when creating leases for an empty lease table to initialize leases for a snapshot of shards specified at `KinesisClientLibConfiguration#initialPositionInStreamExtended`\.

  For more information about `ShardFilter`, see [https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ShardFilter.html](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ShardFilter.html)\.
+ Instead of all workers performing the lease/shard synchronization to keep the lease table up to date with the latest shards in the data stream, a single elected worker leader performs the lease/shard synchronization\.
+ KCL 1\.14 uses the `ChildShards` return parameter of the `GetRecords` and the `SubscribeToShard` APIs to perform lease/shard synchronization that happens at `SHARD_END` for closed shards, allowing a KCL worker to only create leases for the child shards of the shard it finished processing\. For more information, see [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) and [ChildShard](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ChildShard.html)\.
+ With the above changes, the behavior of KCL is moving from the model of all workers learning about all existing shards to the model of workers learning only about the children shards of the shards that each worker owns\. Therefore, in addition to the synchronization that happens during consumer application bootstraping and reshard events, KCL now also performs additional periodic shard/lease scans in order to identify any potential holes in the lease table \(in other words, to learn about all new shards\) to ensure the complete hash range of the data stream is being processed and create leases for them if required\. `PeriodicShardSyncManager` is the component that is responsible for running periodic lease/shard scans\. 

  When `KinesisClientLibConfiguration#shardSyncStrategyType` is set to `ShardSyncStrategyType.SHARD_END`, `PeriodicShardSync leasesRecoveryAuditorInconsistencyConfidenceThreshold` is used to determine the threshold for number of consecutive scans containing holes in the lease table after which to enforce a shard synchronization\. When `KinesisClientLibConfiguration#shardSyncStrategyType` is set to `ShardSyncStrategyType.PERIODIC`, `leasesRecoveryAuditorInconsistencyConfidenceThreshold` is ignored\.

  For more information about `PeriodicShardSyncManager` in KCL 1\.14, see [https://github\.com/awslabs/amazon\-kinesis\-client/blob/v1\.x/src/main/java/com/amazonaws/services/kinesis/clientlibrary/lib/worker/KinesisClientLibConfiguration\.java\#L987\-L999](https://github.com/awslabs/amazon-kinesis-client/blob/v1.x/src/main/java/com/amazonaws/services/kinesis/clientlibrary/lib/worker/KinesisClientLibConfiguration.java#L987-L999)\.

  In KCL 1\.14, new configuration option is available to configure `PeriodicShardSyncManager` in `LeaseManagementConfig`:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/streams/latest/dev/shared-throughput-kcl-consumers.html)

  New CloudWatch metrics are also now emitted to monitor the health of the `PeriodicShardSyncManager`\. For more information, see [PeriodicShardSyncManager](monitoring-with-kcl.md#periodic-task)\.
+ KCL 1\.14 now also supports deferred lease cleanup\. Leases are deleted asynchronously by `LeaseCleanupManager` upon reaching `SHARD_END`, when a shard has either expired past the data stream’s retention period or been closed as the result of a resharding operation\.

  New configuration options are available to configure `LeaseCleanupManager`\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/streams/latest/dev/shared-throughput-kcl-consumers.html)
+ Including an optimization to `KinesisShardSyncer` to only create leases for one layer of shards\.