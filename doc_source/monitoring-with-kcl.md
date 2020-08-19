# Monitoring the Kinesis Client Library with Amazon CloudWatch<a name="monitoring-with-kcl"></a>

The [Kinesis Client Library](https://docs.aws.amazon.com/kinesis/latest/dev/developing-consumers-with-kcl.html) \(KCL\) for Amazon Kinesis Data Streams publishes custom Amazon CloudWatch metrics on your behalf, using the name of your KCL application as the namespace\. You can view these metrics by navigating to the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/) and choosing **Custom Metrics**\. For more information about custom metrics, see [Publish Custom Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/publishingMetrics.html) in the *Amazon CloudWatch User Guide*\.

There is a nominal charge for the metrics uploaded to CloudWatch by the KCL; specifically, *Amazon CloudWatch Custom Metrics* and *Amazon CloudWatch API Requests* charges apply\. For more information, see [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)\.

**Topics**
+ [Metrics and Namespace](#metrics-namespace)
+ [Metric Levels and Dimensions](#metric-levels)
+ [Metric Configuration](#metrics-config)
+ [List of Metrics](#kcl-metrics-list)

## Metrics and Namespace<a name="metrics-namespace"></a>

The namespace that is used to upload metrics is the application name that you specify when you launch the KCL\.

## Metric Levels and Dimensions<a name="metric-levels"></a>

There are two options to control which metrics are uploaded to CloudWatch:

metric levels  
Every metric is assigned an individual level\. When you set a metrics reporting level, metrics with an individual level below the reporting level are not sent to CloudWatch\. The levels are: `NONE`, `SUMMARY`, and `DETAILED`\. The default setting is `DETAILED`; that is, all metrics are sent to CloudWatch\. A reporting level of `NONE` means that no metrics are sent at all\. For information about which levels are assigned to what metrics, see [List of Metrics](#kcl-metrics-list)\.

enabled dimensions  
Every KCL metric has associated dimensions that also get sent to CloudWatch\. `Operation` dimension is always uploaded and cannot be disabled\. By default, the `WorkerIdentifier` dimension is disabled, and only the `Operation` and `ShardId` dimensions are uploaded\.  
For more information about CloudWatch metric dimensions, see the [Dimensions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/cloudwatch_concepts.html#Dimension) section in the Amazon CloudWatch Concepts topic, in the *Amazon CloudWatch User Guide*\.  
When the `WorkerIdentifier` dimension is enabled, if a different value is used for the worker ID property every time a particular KCL worker restarts, new sets of metrics with new `WorkerIdentifier` dimension values are sent to CloudWatch\. If you need the `WorkerIdentifier` dimension value to be the same across specific KCL worker restarts, you must explicitly specify the same worker ID value during initialization for each worker\. Note that the worker ID value for each active KCL worker must be unique across all KCL workers\.

## Metric Configuration<a name="metrics-config"></a>

Metric levels and enabled dimensions can be configured using the KinesisClientLibConfiguration instance, which is passed to Worker when launching the KCL application\. In the MultiLangDaemon case, the `metricsLevel` and `metricsEnabledDimensions` properties can be specified in the \.properties file used to launch the MultiLangDaemon KCL application\.

Metric levels can be assigned one of three values: NONE, SUMMARY, or DETAILED\. Enabled dimensions values must be comma\-separated strings with the list of dimensions that are allowed for the CloudWatch metrics\. The dimensions used by the KCL application are `Operation`, `ShardId`, and `WorkerIdentifier`\.

## List of Metrics<a name="kcl-metrics-list"></a>

The following tables list the KCL metrics, grouped by scope and operation\.

**Topics**
+ [Per\-KCL\-Application Metrics](#kcl-metrics-per-app)
+ [Per\-Worker Metrics](#kcl-metrics-per-worker)
+ [Per\-Shard Metrics](#kcl-metrics-per-shard)

### Per\-KCL\-Application Metrics<a name="kcl-metrics-per-app"></a>

These metrics are aggregated across all KCL workers within the scope of the application, as defined by the Amazon CloudWatch namespace\.

**Topics**
+ [InitializeTask](#init-task)
+ [ShutdownTask](#shutdown-task)
+ [ShardSyncTask](#shard-sync-task)
+ [BlockOnParentTask](#block-parent-task)
+ [PeriodicShardSyncManager](#periodic-task)

#### InitializeTask<a name="init-task"></a>

The `InitializeTask` operation is responsible for initializing the record processor for the KCL application\. The logic for this operation includes getting a shard iterator from Kinesis Data Streams and initializing the record processor\.


| Metric | Description | 
| --- | --- | 
| KinesisDataFetcher\.getIterator\.Success |  Number of successful `GetShardIterator` operations per KCL application\.  Metric level: Detailed Units: Count  | 
| KinesisDataFetcher\.getIterator\.Time |  Time taken per `GetShardIterator` operation for the given KCL application\. Metric level: Detailed Units: Milliseconds  | 
| RecordProcessor\.initialize\.Time |  Time taken by the record processor’s initialize method\. Metric level: Summary Units: Milliseconds  | 
| Success |  Number of successful record processor initializations\.  Metric level: Summary Units: Count  | 
| Time |  Time taken by the KCL worker for the record processor initialization\. Metric level: Summary Units: Milliseconds  | 

#### ShutdownTask<a name="shutdown-task"></a>

The `ShutdownTask` operation initiates the shutdown sequence for shard processing\. This can occur because a shard is split or merged, or when the shard lease is lost from the worker\. In both cases, the record processor `shutdown()` function is invoked\. New shards are also discovered in the case where a shard was split or merged, resulting in the creation of one or two new shards\.


| Metric | Description | 
| --- | --- | 
| CreateLease\.Success |  Number of times that new child shards are successfully added into the KCL application DynamoDB table following parent shard shutdown\. Metric level: Detailed Units: Count  | 
| CreateLease\.Time |  Time taken for adding new child shard information in the KCL application DynamoDB table\. Metric level: Detailed Units: Milliseconds  | 
| UpdateLease\.Success |  Number of successful final checkpoints during the record processor shutdown\. Metric level: Detailed Units: Count  | 
| UpdateLease\.Time |  Time taken by the checkpoint operation during the record processor shutdown\. Metric level: Detailed Units: Milliseconds  | 
| RecordProcessor\.shutdown\.Time |  Time taken by the record processor’s shutdown method\. Metric level: Summary Units: Milliseconds  | 
| Success |  Number of successful shutdown tasks\. Metric level: Summary Units: Count  | 
| Time |  Time taken by the KCL worker for the shutdown task\. Metric level: Summary Units: Milliseconds  | 

#### ShardSyncTask<a name="shard-sync-task"></a>

The `ShardSyncTask` operation discovers changes to shard information for the Kinesis data stream, so new shards can be processed by the KCL application\.


| Metric | Description | 
| --- | --- | 
| CreateLease\.Success |  Number of successful attempts to add new shard information into the KCL application DynamoDB table\. Metric level: Detailed Units: Count  | 
| CreateLease\.Time |  Time taken for adding new shard information in the KCL application DynamoDB table\. Metric level: Detailed Units: Milliseconds  | 
| Success |  Number of successful shard sync operations\. Metric level: Summary Units: Count  | 
| Time |  Time taken for the shard sync operation\. Metric level: Summary Units: Milliseconds  | 

#### BlockOnParentTask<a name="block-parent-task"></a>

If the shard is split or merged with other shards, then new child shards are created\. The `BlockOnParentTask` operation ensures that record processing for the new shards does not start until the parent shards are completely processed by the KCL\.


| Metric | Description | 
| --- | --- | 
| Success |  Number of successful checks for parent shard completion\. Metric level: Summary Units: Count  | 
| Time |  Time taken for parent shards completion\. Metric level: Summary Unit: Milliseconds  | 

#### PeriodicShardSyncManager<a name="periodic-task"></a>

The `PeriodicShardSyncManager` is responsible for examining the data streams that are being processed by the KCL consumer application, identifying data streams with partial leases and handing them off for synchronization\.\.


| Metric | Description | 
| --- | --- | 
| NumStreamsToSync |  The number of data streams \(per AWS account\) being processed by the consumer application that contain partial leases and that must be handed off for synchronization\.  Metric level: Summary Units: Count  | 
| NumStreamsWithPartialLeases |  The number of data streams \(per AWS account\) that the consumer application is processing that contain partial leases\.  Metric level: Summary Units: Count  | 
| Success |  The number of times `PeriodicShardSyncManager` was able to successfully identify partial leases in the data streams that the consumer application is processing\.  Metric level: Summary Units: Count  | 
| Time |  The amount of the time \(in miliseconds\) that the `PeriodicShardSyncManager` takes to examine the data streams that the consumer application is processing, in order to determine which data streams require shard synchronization\.  Metric level: Summary Units: Milliseconds  | 

### Per\-Worker Metrics<a name="kcl-metrics-per-worker"></a>

These metrics are aggregated across all record processors consuming data from a Kinesis data stream, such as an Amazon EC2 instance\.

**Topics**
+ [RenewAllLeases](#renew-leases)
+ [TakeLeases](#take-leases)

#### RenewAllLeases<a name="renew-leases"></a>

The `RenewAllLeases` operation periodically renews shard leases owned by a particular worker instance\. 


| Metric | Description | 
| --- | --- | 
| RenewLease\.Success |  Number of successful lease renewals by the worker\. Metric level: Detailed Units: Count  | 
| RenewLease\.Time |  Time taken by the lease renewal operation\. Metric level: Detailed Units: Milliseconds  | 
| CurrentLeases |  Number of shard leases owned by the worker after all leases are renewed\. Metric level: Summary Units: Count  | 
| LostLeases |  Number of shard leases that were lost following an attempt to renew all leases owned by the worker\. Metric level: Summary Units: Count  | 
| Success |  Number of times lease renewal operation was successful for the worker\. Metric level: Summary Units: Count  | 
| Time |  Time taken for renewing all leases for the worker\. Metric level: Summary Units: Milliseconds  | 

#### TakeLeases<a name="take-leases"></a>

The `TakeLeases` operation balances record processing between all KCL workers\. If the current KCL worker has fewer shard leases than required, it takes shard leases from another worker that is overloaded\.


| Metric | Description | 
| --- | --- | 
| ListLeases\.Success |  Number of times all shard leases were successfully retrieved from the KCL application DynamoDB table\. Metric level: Detailed Units: Count  | 
| ListLeases\.Time |  Time taken to retrieve all shard leases from the KCL application DynamoDB table\. Metric level: Detailed Units: Milliseconds  | 
| TakeLease\.Success |  Number of times the worker successfully took shard leases from other KCL workers\. Metric level: Detailed Units: Count  | 
| TakeLease\.Time |  Time taken to update the lease table with leases taken by the worker\. Metric level: Detailed Units: Milliseconds  | 
| NumWorkers |  Total number of workers, as identified by a specific worker\. Metric level: Summary Units: Count  | 
| NeededLeases |  Number of shard leases that the current worker needs for a balanced shard\-processing load\. Metric level: Detailed Units: Count  | 
| LeasesToTake |  Number of leases that the worker will attempt to take\. Metric level: Detailed Units: Count  | 
| TakenLeases |  Number of leases taken successfully by the worker\. Metric level: Summary Units: Count   | 
| TotalLeases |  Total number of shards that the KCL application is processing\. Metric level: Detailed Units: Count  | 
| ExpiredLeases |  Total number of shards that are not being processed by any worker, as identified by the specific worker\. Metric level: Summary Units: Count  | 
| Success |  Number of times the `TakeLeases` operation successfully completed\. Metric level: Summary Units: Count  | 
| Time |  Time taken by the `TakeLeases` operation for a worker\. Metric level: Summary Units: Milliseconds  | 

### Per\-Shard Metrics<a name="kcl-metrics-per-shard"></a>

These metrics are aggregated across a single record processor\.

#### ProcessTask<a name="process-task"></a>

The `ProcessTask` operation calls [GetRecords](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html) with the current iterator position to retrieve records from the stream and invokes the record processor `processRecords` function\.


| Metric | Description | 
| --- | --- | 
| KinesisDataFetcher\.getRecords\.Success |  Number of successful `GetRecords` operations per Kinesis data stream shard\.  Metric level: Detailed Units: Count  | 
| KinesisDataFetcher\.getRecords\.Time |  Time taken per `GetRecords` operation for the Kinesis data stream shard\. Metric level: Detailed Units: Milliseconds  | 
| UpdateLease\.Success |  Number of successful checkpoints made by the record processor for the given shard\. Metric level: Detailed Units: Count  | 
| UpdateLease\.Time |  Time taken for each checkpoint operation for the given shard\. Metric level: Detailed Units: Milliseconds  | 
| DataBytesProcessed |  Total size of records processed in bytes on each `ProcessTask` invocation\. Metric level: Summary Units: Byte  | 
| RecordsProcessed |  Number of records processed on each `ProcessTask` invocation\. Metric level: Summary Units: Count  | 
| ExpiredIterator |  Number of ExpiredIteratorException received when calling `GetRecords`\. Metric level: Summary Units: Count  | 
| MillisBehindLatest | Time that the current iterator is behind from the latest record \(tip\) in the shard\. This value is less than or equal to the difference in time between the latest record in a response and the current time\. This is a more accurate reflection of how far a shard is from the tip than comparing time stamps in the last response record\. This value applies to the latest batch of records, not an average of all time stamps in each record\.Metric level: SummaryUnits: Milliseconds | 
| RecordProcessor\.processRecords\.Time |  Time taken by the record processor’s `processRecords` method\. Metric level: Summary Units: Milliseconds  | 
| Success |  Number of successful process task operations\. Metric level: Summary Units: Count  | 
| Time |  Time taken for the process task operation\. Metric level: Summary Units: Milliseconds  | 