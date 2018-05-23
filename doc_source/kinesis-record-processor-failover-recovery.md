# Recovering from Failures in Amazon Kinesis Data Streams<a name="kinesis-record-processor-failover-recovery"></a>

Failure can occur at the following levels when you use an Amazon Kinesis Data Streams application to process data from a stream:
+ A record processor could fail
+ A worker could fail, or the instance of the application that instantiated the worker could fail 
+ An EC2 instance that is hosting one or more instances of the application could fail

## Record Processor Failure<a name="kinesis-record-processor-failure-processor"></a>

The worker invokes record processor methods using Java [ExecutorService](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html) tasks\. If a task fails, the worker retains control of the shard that the record processor was processing\. The worker starts a new record processor task to process that shard\. For more information, see [Read Throttling](kinesis-record-processor-additional-considerations.md#kinesis-record-processor-read-throttling)\. 

## Worker or Application Failure<a name="kinesis-record-processor-failure-worker"></a>

If a worker — or an instance of the Amazon Kinesis Data Streams application — fails, you should detect and handle the situation\. For example, if the `Worker.run` method throws an exception, you should catch and handle it\. 

If the application itself fails, you should detect this and restart it\. When the application starts up, it instantiates a new worker, which in turn instantiates new record processors that are automatically assigned shards to process\. These could be the same shards that these record processors were processing before the failure, or shards that are new to these processors\. 

If the worker or application fails but you do not detect the failure, and there are other instances of the application running on other EC2 instances, the workers on these instances handle the failure: they create additional record processors to process the shards that are no longer being processed by the failed worker\. The load on these other EC2 instances increases accordingly\. 

The scenario described here assumes that although the worker or application has failed, the hosting EC2 instance is still running and is therefore not restarted by an Auto Scaling group\. 

## Amazon EC2 Instance Failure<a name="kinesis-record-processor-failure-instance"></a>

We recommend that you run the EC2 instances for your application in an Auto Scaling group\. This way, if one of the EC2 instances fails, the Auto Scaling group automatically launches a new instance to replace it\. You should configure the instances to launch your Amazon Kinesis Data Streams application at startup\.