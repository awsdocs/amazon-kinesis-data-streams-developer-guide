# Consumers for Amazon Kinesis Streams<a name="amazon-kinesis-consumers"></a>

A consumer gets data records from Kinesis streams\. A consumer, known as a *Amazon Kinesis Streams application*, processes the data records from a stream\.

**Important**  
[Changing the Data Retention Period](kinesis-extended-retention.md)

**Note**  
To save stream records directly to storage services such as Amazon S3, Amazon Redshift, or Amazon Elasticsearch Service, you can use a Kinesis Firehose delivery stream instead of creating a consumer application\. For more information, see [Creating an Amazon Kinesis Firehose Delivery Stream](http://docs.aws.amazon.com/firehose/latest/dev/basic-create.html)\.

Each consumer reads from a particular shard, using a shard iterator\. A *shard iterator* represents the position in the stream from which the consumer reads\. When they start reading from a stream, consumers get a shard iterator, which can be used to change where the consumers read from the stream\. When the consumer performs a read operation, it receives a batch of data records based on the position specified by the shard iterator\.

Each consumer must have a unique name that is scoped to the AWS account and region used by the application\. This name is used as a name for the control table in Amazon DynamoDB and the namespace for Amazon CloudWatch metrics\. When your application starts up, it creates an Amazon DynamoDB table to store the application state, connects to the specified stream, and then starts consuming data from the stream\. You can view the Kinesis Streams metrics using the CloudWatch console\.

You can deploy the consumer to an EC2 instance by adding to one of your AMIs\. You can scale the consumer by running it on multiple EC2 instances under an Auto Scaling group\. Using an Auto Scaling group helps automatically start new instances in the event of an EC2 instance failure and can also elastically scale the number of instances as the load on the application changes over time\. Auto Scaling groups ensure that a certain number of EC2 instances are always running\. To trigger scaling events in the Auto Scaling group, you can specify metrics such as CPU and memory utilization to scale up or down the number of EC2 instances processing data from the stream\. For more information, see the [Auto Scaling User Guide](http://docs.aws.amazon.com/autoscaling/latest/userguide/)\.

You can use the Kinesis Client Library \(KCL\) to simplify parallel processing of the stream by a fleet of workers running on a fleet of EC2 instances\. The KCL simplifies writing code to read from the shards in the stream and ensures that there is a worker allocated to every shard in the stream\. The KCL also provides help with fault tolerance by providing checkpointing capabilities\. The best way to get started with the KCL is to review the samples in [Developing Amazon Kinesis Streams Consumers Using the Kinesis Client Library](developing-consumers-with-kcl.md)\.