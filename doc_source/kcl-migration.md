# Migrating Consumers from KCL 1\.x to KCL 2\.x<a name="kcl-migration"></a>

This topic explains the differences between versions 1\.x and 2\.x of the Kinesis Client Library \(KCL\)\. It also shows you how to migrate your consumer from version 1\.x to version 2\.x of the KCL\. After you migrate your client, it will start processing records from the last checkpointed location\.

Version 2\.0 of the KCL introduces the following interface changes:


**KCL Interface Changes**  

| KCL 1\.x Interface | KCL 2\.0 Interface | 
| --- | --- | 
| com\.amazonaws\.services\.kinesis\.clientlibrary\.interfaces\.v2\.IRecordProcessor | software\.amazon\.kinesis\.processor\.ShardRecordProcessor | 
| com\.amazonaws\.services\.kinesis\.clientlibrary\.interfaces\.v2\.IRecordProcessorFactory | software\.amazon\.kinesis\.processor\.ShardRecordProcessorFactory | 
| com\.amazonaws\.services\.kinesis\.clientlibrary\.interfaces\.v2\.IShutdownNotificationAware | Folded into software\.amazon\.kinesis\.processor\.ShardRecordProcessor | 

**Topics**
+ [Migrating the Record Processor](#recrod-processor-migration)
+ [Migrating the Record Processor Factory](#recrod-processor-factory-migration)
+ [Migrating the Worker](#worker-migration)
+ [Configuring the Amazon Kinesis Client](#client-configuration)
+ [Idle Time Removal](#idle-time-removal)
+ [Client Configuration Removals](#client-configuration-removals)

## Migrating the Record Processor<a name="recrod-processor-migration"></a>

The following example shows a record processor implemented for KCL 1\.x:

```
package com.amazonaws.kcl;

import com.amazonaws.services.kinesis.clientlibrary.exceptions.InvalidStateException;
import com.amazonaws.services.kinesis.clientlibrary.exceptions.ShutdownException;
import com.amazonaws.services.kinesis.clientlibrary.interfaces.IRecordProcessorCheckpointer;
import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessor;
import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IShutdownNotificationAware;
import com.amazonaws.services.kinesis.clientlibrary.lib.worker.ShutdownReason;
import com.amazonaws.services.kinesis.clientlibrary.types.InitializationInput;
import com.amazonaws.services.kinesis.clientlibrary.types.ProcessRecordsInput;
import com.amazonaws.services.kinesis.clientlibrary.types.ShutdownInput;

public class TestRecordProcessor implements IRecordProcessor, IShutdownNotificationAware {
    @Override
    public void initialize(InitializationInput initializationInput) {
        //
        // Setup record processor
        //
    }

    @Override
    public void processRecords(ProcessRecordsInput processRecordsInput) {
        //
        // Process records, and possibly checkpoint
        //
    }

    @Override
    public void shutdown(ShutdownInput shutdownInput) {
        if (shutdownInput.getShutdownReason() == ShutdownReason.TERMINATE) {
            try {
                shutdownInput.getCheckpointer().checkpoint();
            } catch (ShutdownException | InvalidStateException e) {
                throw new RuntimeException(e);
            }
        }
    }

    @Override
    public void shutdownRequested(IRecordProcessorCheckpointer checkpointer) {
        try {
            checkpointer.checkpoint();
        } catch (ShutdownException | InvalidStateException e) {
            //
            // Swallow exception
            //
            e.printStackTrace();
        }
    }
}
```

**To migrate the record processor class**

1. Change the interfaces from `com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessor` and `com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IShutdownNotificationAware` to `software.amazon.kinesis.processor.ShardRecordProcessor`, as follows:

   ```
   // import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessor;
   // import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IShutdownNotificationAware;
   import software.amazon.kinesis.processor.ShardRecordProcessor;
   
   // public class TestRecordProcessor implements IRecordProcessor, IShutdownNotificationAware {
   public class TestRecordProcessor implements ShardRecordProcessor {
   ```

1. Update the `import` statements for the `initialize` and `processRecords` methods\.

   ```
   // import com.amazonaws.services.kinesis.clientlibrary.types.InitializationInput;
   import software.amazon.kinesis.lifecycle.events.InitializationInput;
   
   //import com.amazonaws.services.kinesis.clientlibrary.types.ProcessRecordsInput;
   import software.amazon.kinesis.lifecycle.events.ProcessRecordsInput;
   ```

1. Replace the `shutdown` method with the following new methods: `leaseLost`, `shardEnded`, and `shutdownRequested`\.

   ```
   //    @Override
   //    public void shutdownRequested(IRecordProcessorCheckpointer checkpointer) {
   //        //
   //        // This is moved to shardEnded(...)
   //        //
   //        try {
   //            checkpointer.checkpoint();
   //        } catch (ShutdownException | InvalidStateException e) {
   //            //
   //            // Swallow exception
   //            //
   //            e.printStackTrace();
   //        }
   //    }
   
       @Override
       public void leaseLost(LeaseLostInput leaseLostInput) {
   
       }
   
       @Override
       public void shardEnded(ShardEndedInput shardEndedInput) {
           try {
               shardEndedInput.checkpointer().checkpoint();
           } catch (ShutdownException | InvalidStateException e) {
               //
               // Swallow the exception
               //
               e.printStackTrace();
           }
       }
   
   //    @Override
   //    public void shutdownRequested(IRecordProcessorCheckpointer checkpointer) {
   //        //
   //        // This is moved to shutdownRequested(ShutdownReauestedInput)
   //        //
   //        try {
   //            checkpointer.checkpoint();
   //        } catch (ShutdownException | InvalidStateException e) {
   //            //
   //            // Swallow exception
   //            //
   //            e.printStackTrace();
   //        }
   //    }
   
       @Override
       public void shutdownRequested(ShutdownRequestedInput shutdownRequestedInput) {
           try {
               shutdownRequestedInput.checkpointer().checkpoint();
           } catch (ShutdownException | InvalidStateException e) {
               //
               // Swallow the exception
               //
               e.printStackTrace();
           }
       }
   ```

The following is the updated version of the record processor class\.

```
package com.amazonaws.kcl;

import software.amazon.kinesis.exceptions.InvalidStateException;
import software.amazon.kinesis.exceptions.ShutdownException;
import software.amazon.kinesis.lifecycle.events.InitializationInput;
import software.amazon.kinesis.lifecycle.events.LeaseLostInput;
import software.amazon.kinesis.lifecycle.events.ProcessRecordsInput;
import software.amazon.kinesis.lifecycle.events.ShardEndedInput;
import software.amazon.kinesis.lifecycle.events.ShutdownRequestedInput;
import software.amazon.kinesis.processor.ShardRecordProcessor;

public class TestRecordProcessor implements ShardRecordProcessor {
    @Override
    public void initialize(InitializationInput initializationInput) {
        
    }

    @Override
    public void processRecords(ProcessRecordsInput processRecordsInput) {
        
    }

    @Override
    public void leaseLost(LeaseLostInput leaseLostInput) {
        
    }

    @Override
    public void shardEnded(ShardEndedInput shardEndedInput) {
        try {
            shardEndedInput.checkpointer().checkpoint();
        } catch (ShutdownException | InvalidStateException e) {
            //
            // Swallow the exception
            //
            e.printStackTrace();
        }
    }

    @Override
    public void shutdownRequested(ShutdownRequestedInput shutdownRequestedInput) {
        try {
            shutdownRequestedInput.checkpointer().checkpoint();
        } catch (ShutdownException | InvalidStateException e) {
            //
            // Swallow the exception
            //
            e.printStackTrace();
        }
    }
}
```

## Migrating the Record Processor Factory<a name="recrod-processor-factory-migration"></a>

The record processor factory is responsible for creating record processors when a lease is acquired\. The following is an example of a KCL 1\.x factory\.

```
package com.amazonaws.kcl;

import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessor;
import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessorFactory;

public class TestRecordProcessorFactory implements IRecordProcessorFactory {
    @Override
    public IRecordProcessor createProcessor() {
        return new TestRecordProcessor();
    }
}
```

**To migrate the record processor factory**

1. Change the implemented interface from `com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessorFactory` to `software.amazon.kinesis.processor.ShardRecordProcessorFactory`, as follows\.

   ```
   // import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessor;
   import software.amazon.kinesis.processor.ShardRecordProcessor;
   
   // import com.amazonaws.services.kinesis.clientlibrary.interfaces.v2.IRecordProcessorFactory;
   import software.amazon.kinesis.processor.ShardRecordProcessorFactory;
   
   // public class TestRecordProcessorFactory implements IRecordProcessorFactory {
   public class TestRecordProcessorFactory implements ShardRecordProcessorFactory {
   ```

1. Change the return signature for `createProcessor`\.

   ```
   // public IRecordProcessor createProcessor() {
   public ShardRecordProcessor shardRecordProcessor() {
   ```

The following is an example of the record processor factory in 2\.0:

```
package com.amazonaws.kcl;

import software.amazon.kinesis.processor.ShardRecordProcessor;
import software.amazon.kinesis.processor.ShardRecordProcessorFactory;

public class TestRecordProcessorFactory implements ShardRecordProcessorFactory {
    @Override
    public ShardRecordProcessor shardRecordProcessor() {
        return new TestRecordProcessor();
    }
}
```

## Migrating the Worker<a name="worker-migration"></a>

In version 2\.0 of the KCL, a new class, called `Scheduler`, replaces the `Worker` class\. The following is an example of a KCL 1\.x worker\.

```
final KinesisClientLibConfiguration config = new KinesisClientLibConfiguration(...)
final IRecordProcessorFactory recordProcessorFactory = new RecordProcessorFactory();
final Worker worker = new Worker.Builder()
    .recordProcessorFactory(recordProcessorFactory)
    .config(config)
    .build();
```

**To migrate the worker**

1. Change the `import` statement for the `Worker` class to the import statements for the `Scheduler` and `ConfigsBuilder` classes\.

   ```
   // import com.amazonaws.services.kinesis.clientlibrary.lib.worker.Worker;
   import software.amazon.kinesis.coordinator.Scheduler;
   import software.amazon.kinesis.common.ConfigsBuilder;
   ```

1. Create the `ConfigsBuilder` and a `Scheduler` as shown in the following example\.

   It is recommended that you use `KinesisClientUtil` to create `KinesisAsyncClient` and to configure `maxConcurrency` in `KinesisAsyncClient`\.
**Important**  
The Amazon Kinesis Client might see significantly increased latency, unless you configure `KinesisAsyncClient` to have a `maxConcurrency` high enough to allow all leases plus additional usages of `KinesisAsyncClient`\.

   ```
   import java.util.UUID;
   
   import software.amazon.awssdk.regions.Region;
   import software.amazon.awssdk.services.dynamodb.DynamoDbAsyncClient;
   import software.amazon.awssdk.services.cloudwatch.CloudWatchAsyncClient;
   import software.amazon.awssdk.services.kinesis.KinesisAsyncClient;
   import software.amazon.kinesis.common.ConfigsBuilder;
   import software.amazon.kinesis.common.KinesisClientUtil;
   import software.amazon.kinesis.coordinator.Scheduler;
   
   ...
   
   Region region = Region.AP_NORTHEAST_2;
   KinesisAsyncClient kinesisClient = KinesisClientUtil.createKinesisAsyncClient(KinesisAsyncClient.builder().region(region));
   DynamoDbAsyncClient dynamoClient = DynamoDbAsyncClient.builder().region(region).build();
   CloudWatchAsyncClient cloudWatchClient = CloudWatchAsyncClient.builder().region(region).build();
   
   ConfigsBuilder configsBuilder = new ConfigsBuilder(streamName, applicationName, kinesisClient, dynamoClient, cloudWatchClient, UUID.randomUUID().toString(), new SampleRecordProcessorFactory());
   
   Scheduler scheduler = new Scheduler(
       configsBuilder.checkpointConfig(),
       configsBuilder.coordinatorConfig(),
       configsBuilder.leaseManagementConfig(),
       configsBuilder.lifecycleConfig(),
       configsBuilder.metricsConfig(),
       configsBuilder.processorConfig(),
       configsBuilder.retrievalConfig()
       );
   ```

## Configuring the Amazon Kinesis Client<a name="client-configuration"></a>

With the 2\.0 release of the Kinesis Client Library, the configuration of the client moved from a single configuration class \(`KinesisClientLibConfiguration`\) to six configuration classes\. The following table describes the migration\.


**Configuration Fields and Their New Classes**  

| Original Field | New Configuration Class | Description | 
| --- | --- | --- | 
| applicationName | ConfigsBuilder | The name for this the KCL application\. Used as the default for the tableName and consumerName\. | 
| tableName | ConfigsBuilder | Allows overriding the table name used for the Amazon DynamoDB lease table\. | 
| streamName | ConfigsBuilder | The name of the stream that this application processes records from\. | 
| kinesisEndpoint | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| dynamoDBEndpoint | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| initialPositionInStreamExtended | RetrievalConfig | The location in the shard from which the KCL begins fetching records, starting with the application's initial run\. | 
| kinesisCredentialsProvider | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| dynamoDBCredentialsProvider | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| cloudWatchCredentialsProvider | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| failoverTimeMillis | LeaseManagementConfig | The number of milliseconds that must pass before you can consider a lease owner to have failed\. | 
| workerIdentifier | ConfigsBuilder | A unique identifier that represents this instantiation of the application processor\. This must be unique\. | 
| shardSyncIntervalMillis | LeaseManagementConfig | The time between shard sync calls\. | 
| maxRecords | PollingConfig | Allows setting the maximum number of records that Kinesis returns\. | 
| idleTimeBetweenReadsInMillis | CoordinatorConfig | This option has been removed\. See Idle Time Removal\. | 
| callProcessRecordsEvenForEmptyRecordList | ProcessorConfig | When set, the record processor is called even when no records were provided from Kinesis\. | 
| parentShardPollIntervalMillis | CoordinatorConfig | How often a record processor should poll to see if the parent shard has been completed\. | 
| cleanupLeasesUponShardCompletion | LeaseManagementConfig | When set, leases are removed as soon as the child leases have started processing\. | 
| ignoreUnexpectedChildShards | LeaseManagementConfig | When set, child shards that have an open shard are ignored\. This is primarily for DynamoDB Streams\. | 
| kinesisClientConfig | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| dynamoDBClientConfig | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| cloudWatchClientConfig | ConfigsBuilder | This option has been removed\. See Client Configuration Removals\. | 
| taskBackoffTimeMillis | LifecycleConfig | The time to wait to retry failed tasks\. | 
| metricsBufferTimeMillis | MetricsConfig | Controls CloudWatch metric publishing\. | 
| metricsMaxQueueSize | MetricsConfig | Controls CloudWatch metric publishing\. | 
| metricsLevel | MetricsConfig | Controls CloudWatch metric publishing\. | 
| metricsEnabledDimensions | MetricsConfig | Controls CloudWatch metric publishing\. | 
| validateSequenceNumberBeforeCheckpointing | CheckpointConfig | This option has been removed\. See Checkpoint Sequence Number Validation\. | 
| regionName | ConfigsBuilder | This option has been removed\. See Client Configuration Removal\. | 
| maxLeasesForWorker | LeaseManagementConfig | The maximum number of leases a single instance of the application should accept\. | 
| maxLeasesToStealAtOneTime | LeaseManagementConfig | The maximum number of leases an application should attempt to steal at one time\. | 
| initialLeaseTableReadCapacity | LeaseManagementConfig | The DynamoDB read IOPs that is used if the Kinesis Client Library needs to create a new DynamoDB lease table\. | 
| initialLeaseTableWriteCapacity | LeaseManagementConfig | The DynamoDB read IOPs that is used if the Kinesis Client Library needs to create a new DynamoDB lease table\. | 
| initialPositionInStreamExtended | LeaseManagementConfig | The initial position in the stream that the application should start at\. This is only used during initial lease creation\. | 
| skipShardSyncAtWorkerInitializationIfLeasesExist | CoordinatorConfig | Disable synchronizing shard data if the lease table contains existing leases\. TODO: KinesisEco\-438 | 
| shardPrioritization | CoordinatorConfig | Which shard prioritization to use\. | 
| shutdownGraceMillis | N/A | This option has been removed\. See MultiLang Removals\. | 
| timeoutInSeconds | N/A | This option has been removed\. See MultiLang Removals\. | 
| retryGetRecordsInSeconds | PollingConfig | Configures the delay between GetRecords attempts for failures\. | 
| maxGetRecordsThreadPool | PollingConfig | The thread pool size used for GetRecords\. | 
| maxLeaseRenewalThreads | LeaseManagementConfig | Controls the size of the lease renewer thread pool\. The more leases that your application could take, the larger this pool should be\. | 
| recordsFetcherFactory | PollingConfig | Allows replacing the factory used to create fetchers that retrieve from streams\. | 
| logWarningForTaskAfterMillis | LifecycleConfig | How long to wait before a warning is logged if a task hasn't completed\. | 
| listShardsBackoffTimeInMillis | RetrievalConfig | The number of milliseconds to wait between calls to ListShards when failures occur\. | 
| maxListShardsRetryAttempts | RetrievalConfig | The maximum number of times that ListShards retries before giving up\. | 

## Idle Time Removal<a name="idle-time-removal"></a>

In the 1\.x version of the KCL, the `idleTimeBetweenReadsInMillis` corresponded to two quantities: 
+ The amount of time between task dispatching checks\. You can now configure this time between tasks by setting `CoordinatorConfig#shardConsumerDispatchPollIntervalMillis`\.
+ The amount of time to sleep when no records were returned from Kinesis Data Streams\. In version 2\.0, in enhanced fan\-out records are pushed from their respective retriever\. Activity on the shard consumer only occurs when a pushed request arrives\. 

## Client Configuration Removals<a name="client-configuration-removals"></a>

In version 2\.0, the KCL no longer creates clients\. It depends on the user to supply a valid client\. With this change, all configuration parameters that controlled client creation have been removed\. If you need these parameters, you can set them on the clients before providing the clients to `ConfigsBuilder`\.


****  

| Removed Field | Equivalent Configuration | 
| --- | --- | 
| kinesisEndpoint | Configure the SDK KinesisAsyncClient with preferred endpoint: KinesisAsyncClient\.builder\(\)\.endpointOverride\(URI\.create\("https://<kinesis endpoint>"\)\)\.build\(\)\. | 
| dynamoDBEndpoint | Configure the SDK DynamoDbAsyncClient with preferred endpoint: DynamoDbAsyncClient\.builder\(\)\.endpointOverride\(URI\.create\("https://<dynamodb endpoint>"\)\)\.build\(\)\. | 
| kinesisClientConfig | Configure the SDK KinesisAsyncClient with the needed configuration: KinesisAsyncClient\.builder\(\)\.overrideConfiguration\(<your configuration>\)\.build\(\)\. | 
| dynamoDBClientConfig | Configure the SDK DynamoDbAsyncClient with the needed configuration: DynamoDbAsyncClient\.builder\(\)\.overrideConfiguration\(<your configuration>\)\.build\(\)\. | 
| cloudWatchClientConfig | Configure the SDK CloudWatchAsyncClient with the needed configuration: CloudWatchAsyncClient\.builder\(\)\.overrideConfiguration\(<your configuration>\)\.build\(\)\. | 
| regionName | Configure the SDK with the preferred Region\. This is the same for all SDK clients\. For example, KinesisAsyncClient\.builder\(\)\.region\(Region\.US\_WEST\_2\)\.build\(\)\. | 