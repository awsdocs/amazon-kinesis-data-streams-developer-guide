# Resilience in Amazon Kinesis Data Streams<a name="disaster-recovery-resiliency"></a>

The AWS global infrastructure is built around AWS Regions and Availability Zones\. AWS Regions provide multiple physically separated and isolated Availability Zones, which are connected with low\-latency, high\-throughput, and highly redundant networking\. With Availability Zones, you can design and operate applications and databases that automatically fail over between Availability Zones without interruption\. Availability Zones are more highly available, fault tolerant, and scalable than traditional single or multiple data center infrastructures\. 

For more information about AWS Regions and Availability Zones, see [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)\.

In addition to the AWS global infrastructure, Kinesis Data Streams offers several features to help support your data resiliency and backup needs\.

## Disaster Recovery in Amazon Kinesis Data Streams<a name="disaster-recovery"></a>

Failure can occur at the following levels when you use an Amazon Kinesis Data Streams application to process data from a stream:
+ A record processor could fail
+ A worker could fail, or the instance of the application that instantiated the worker could fail 
+ An EC2 instance that is hosting one or more instances of the application could fail

### Record Processor Failure<a name="kinesis-record-processor-failure-processor"></a>

The worker invokes record processor methods using Java [ExecutorService](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html) tasks\. If a task fails, the worker retains control of the shard that the record processor was processing\. The worker starts a new record processor task to process that shard\. For more information, see [Read Throttling](kinesis-record-processor-additional-considerations.md#kinesis-record-processor-read-throttling)\. 

### Worker or Application Failure<a name="kinesis-record-processor-failure-worker"></a>

If a worker—or an instance of the Amazon Kinesis Data Streams application—fails, you should detect and handle the situation\. For example, if the `Worker.run` method throws an exception, you should catch and handle it\. 

If the application itself fails, you should detect this and restart it\. When the application starts up, it instantiates a new worker, which in turn instantiates new record processors that are automatically assigned shards to process\. These could be the same shards that these record processors were processing before the failure, or shards that are new to these processors\. 

In a situation where the worker or application fails, the failure isn't detected, and there are other instances of the application running on other EC2 instances, workers on these other instances handle the failure\. They create additional record processors to process the shards that are no longer being processed by the failed worker\. The load on these other EC2 instances increases accordingly\.

The scenario described here assumes that although the worker or application has failed, the hosting EC2 instance is still running and is therefore not restarted by an Auto Scaling group\. 

### Amazon EC2 Instance Failure<a name="kinesis-record-processor-failure-instance"></a>

We recommend that you run the EC2 instances for your application in an Auto Scaling group\. This way, if one of the EC2 instances fails, the Auto Scaling group automatically launches a new instance to replace it\. You should configure the instances to launch your Amazon Kinesis Data Streams application at startup\.