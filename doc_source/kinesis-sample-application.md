# Tutorial: Visualizing Web Traffic Using Amazon Kinesis Data Streams<a name="kinesis-sample-application"></a>

This tutorial helps you get started using Amazon Kinesis Data Streams by providing an introduction to key Kinesis Data Streams constructs; specifically [streams](amazon-kinesis-streams.md), [data producers](amazon-kinesis-producers.md), and [data consumers](amazon-kinesis-consumers.md)\. The tutorial uses a sample application based upon a common use case of real\-time data analytics, as introduced in [What Is Amazon Kinesis Data Streams?](introduction.md)\.

The web application for this sample uses a simple JavaScript application to poll the DynamoDB table used to store the results of the Top\-N analysis over a slide window\. The application takes this data and creates a visualization of the results\.

## Kinesis Data Streams Data Visualization Sample Application<a name="kinesis-sample-application-tutorial"></a>

The data visualization sample application for this tutorial demonstrates how to use Kinesis Data Streams for real\-time data ingestion and analysis\. The sample application creates a data producer that puts simulated visitor counts from various URLs into a Kinesis data stream\. The stream durably stores these data records in the order they are received\. The data consumer gets these records from the stream, and then calculates how many visitors originated from a particular URL\. Finally, a simple web application polls the results in real time to provide a visualization of the calculations\.

The sample application demonstrates the common stream processing use\-case of performing a sliding window analysis over a 10\-second period\. The data displayed in the above visualization reflects the results of the sliding window analysis of the stream as a continuously updated graph\. In addition, the data consumer performs Top\-K analysis over the data stream to compute the top three referrers by count, which is displayed in the table immediately below the graph and updated every two seconds\. 

To get you started quickly, the sample application uses [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/)\. AWS CloudFormation allows you to create templates to describe the AWS resources and any associated dependencies or runtime parameters required to run your application\. The sample application uses a template to create all the necessary resources quickly, including producer and consumer applications running on an Amazon EC2 instance and a table in Amazon DynamoDB to store the aggregate record counts\.

**Note**  
After the sample application starts, it incurs nominal charges for Kinesis Data Streams usage\. Where possible, the sample application uses resources that are eligible for the [AWS Free Tier](https://aws.amazon.com/free)\. When you are finished with this tutorial, delete your AWS resources to stop incurring charges\. For more information, see [Step 3: Delete Sample Application](#kinesis-gs-deleting-sample-resources)\.

## Prerequisites<a name="kinesis-sample-application-prerequisites"></a>

This tutorial helps you set up, run, and view the results of the Kinesis Data Streams data visualization sample application\. To get started with the sample application, you first need to do the following:
+ Set up a computer with a working Internet connection\.
+ Sign up for an AWS account\.
+ Additionally, read through the introductory sections to gain a high\-level understanding of [streams](amazon-kinesis-streams.md), [data producers](amazon-kinesis-producers.md), and [data consumers](amazon-kinesis-consumers.md)\.

## Step 1: Start the Sample Application<a name="kinesis-gs-run-the-sample-app"></a>

Start the sample application using a AWS CloudFormation template provided by AWS\. The sample application has a stream writer that randomly generates records and sends them to an Kinesis data stream, a data consumer that counts the number of HTTPS requests to a resource, and a web application that displays the outputs of the stream processing data as a continuously updated graph\.

**To start the application**

1. Open the [AWS CloudFormation template](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=KinesisDataVisSampleApp&templateURL=https:%2F%2Fs3.amazonaws.com%2Fkinesis-demo-bucket%2Famazon-kinesis-data-visualization-sample%2Fkinesis-data-vis-sample-app.template) for this tutorial\.

1. On the **Select Template** page, the URL for the template is provided\. Choose **Next**\.

1. On the **Specify Details** page, note that the default instance type is `t2.micro`\. However, T2 instances require a VPC\. If your AWS account does not have a default VPC in your region, you must change **InstanceType** another instance type, such as `m3.medium`\. Choose **Next**\.

1. On the **Options** page, you can optionally type a tag key and tag value\. This tag will be added to the resources created by the template, such as the EC2 instance\. Choose **Next**\.

1. On the **Review page**, select **I acknowledge that this template might cause AWS CloudFormation to create IAM resources**, and then choose **Create**\.

Initially, you should see a stack named **KinesisDataVisSample** with the status `CREATE_IN_PROGRESS`\. The stack can take several minutes to create\. When the status is `CREATE_COMPLETE`, continue to the next step\. If the status does not update, refresh the page\.

## Step 2: View the Components of the Sample Application<a name="kinesis-gs-viewing-results"></a>

**Topics**
+ [Kinesis Data Stream](#kinesis-gs-view-stream)
+ [Data Producer](#kinesis-gs-view-data-producer)
+ [Data Consumer](#kinesis-gs-view-data-consumer)

### Kinesis Data Stream<a name="kinesis-gs-view-stream"></a>

A [stream](amazon-kinesis-streams.md) has the ability to ingest data in real\-time from a large number of producers, durably store the data, and provide the data to multiple consumers\. A stream represents an ordered sequence of data records\. When you create a stream, you must specify a stream name and a shard count\. A stream consists of one or more shards; each shard is a group of data records\.

AWS CloudFormation automatically creates the stream for the sample application\. [This section](https://github.com/awslabs/amazon-kinesis-data-visualization-sample/blob/master/src/main/static-content/cloudformation/kinesis-data-vis-sample-app.template#L74-80) of the AWS CloudFormation template shows the parameters used in the [CreateStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_CreateStream.html) operation\.<a name="kinesis-gs-viewing-results-stack"></a>

**To view the stack details**

1. Select the **KinesisDataVisSample** stack\.

1. On the **Outputs** tab, choose the link in **URL**\. The form of the URL should be similar to the following: http://ec2\-xx\-xx\-xx\-xx\.compute\-1\.amazonaws\.com\.

1. It takes about 10 minutes for the application stack to be created and for meaningful data to show up in the data analysis graph\. The real\-time data analysis graph appears on a separate page, titled **Kinesis Data Streams Data Visualization Sample**\. It displays the number of requests sent by the referring URL over a 10 second span, and the chart is updated every 1 second\. The span of the graph is the last 2 minutes\.  
![\[Data visualization for the sample application\]](http://docs.aws.amazon.com/streams/latest/dev/images/data_visualization.png)<a name="kinesis-gs-viewing-results-stream"></a>

**To view the stream details**

1. Open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Select the stream whose name has the following form: `KinesisDataVisSampleApp-KinesisStream-[randomString]`\.

1. Choose the name of the stream to view the stream details\.

1. The graphs display the activity of the data producer putting records into the stream and the data consumer getting data from the stream\.  
![\[Stream monitoring graphs for the sample application\]](http://docs.aws.amazon.com/streams/latest/dev/images/sample_graphs.png)

### Data Producer<a name="kinesis-gs-view-data-producer"></a>

A [data producer](amazon-kinesis-producers.md) submits data records to the Kinesis data stream\. To put data into the stream, producers call the [PutRecord](http://docs.aws.amazon.com/kinesis/latest/APIReference//API_PutRecord.html) operation on a stream\. 

Each `PutRecord` call requires the stream name, partition key, and the data record that the producer is adding to the stream\. The stream name determines the stream in which the record will reside\. The partition key is used to determine the shard in the stream that the data record will be added to\.

Which partition key you use depends on your application logic\. The number of partition keys should be much greater than the number of shards\. in most cases\. A high number of partition keys relative to shards allows the stream to distribute data records evenly across the shards in your stream\. 

The data producer uses six popular URLs as a partition key for each record put into the two\-shard stream\. These URLs represent simulated page visits\. [Rows 99\-132](https://github.com/awslabs/amazon-kinesis-data-visualization-sample/blob/master/src/main/java/com/amazonaws/services/kinesis/samples/datavis/producer/HttpReferrerKinesisPutter.java#L99-132) of the HttpReferrerKinesisPutter code send the data to Kinesis Data Streams\. The three required parameters are set before calling `PutRecord`\. The partition key is set using `pair.getResource`, which randomly selects one of the six URLs created in [rows 85\-92](https://github.com/awslabs/amazon-kinesis-data-visualization-sample/blob/d7fbcb994caad606635414b01c4eeada15c04f0b/src/main/java/com/amazonaws/services/kinesis/samples/datavis/HttpReferrerStreamWriter.java#L85-92) of the HttpReferrerStreamWriter code\.

A data producer can be anything that puts data to Kinesis Data Streams, such as an EC2 instance, client browser, or mobile device\. The sample application uses an EC2 instance for its data producer as well as its data consumer; whereas, most real\-life scenarios would have separate EC2 instances for each component of the application\. You can view EC2 instance data from the sample application by following the instructions below\.<a name="kinesis-gs-viewing-results-instance-data"></a>

**To view the instance data in the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Instances**\.

1. Select the instance created for the sample application\. If you aren't sure which instance this is, it has a security group with a name that starts with **KinesisDataVisSample**\.

1. On the **Monitoring** tab, you'll see the resource usage of the sample application's data producer and data consumer\.

### Data Consumer<a name="kinesis-gs-view-data-consumer"></a>

[Data consumers](amazon-kinesis-consumers.md) retrieve and process data records from shards in a Kinesis data stream\. Each consumer reads data from a particular shard\. Consumers retrieve data from a shard using the [GetShardIterator](http://docs.aws.amazon.com/kinesis/latest/APIReference//API_GetShardIterator.html) and [GetRecords](http://docs.aws.amazon.com/kinesis/latest/APIReference//API_GetRecords.html) operations\.

A shard iterator represents the position of the stream and shard from which the consumer will read\. A consumer gets a shard iterator when it starts reading from a stream or changes the position from which it reads records from a stream\. To get a shard iterator, you must provide a stream name, shard ID, and shard iterator type\. The shard iterator type allows the consumer to specify where in the stream it would like to start reading from, such as from the start of the stream where the data is arriving in real\-time\. The stream returns the records in a batch, whose size you can control using the optional limit parameter\.

The data consumer creates a table in DynamoDB to maintain state information \(such as checkpoints and worker\-shard mapping\) for the application\. Each application has its own DynamoDB table\.

The data consumer counts visitor requests from each particular URL over the last two seconds\. This type of real\-time application employs Top\-N analysis over a sliding window\. In this case, the Top\-N are the top three pages by visitor requests and the sliding window is two seconds\. This is a common processing pattern that is demonstrative of real\-world data analyses using Kinesis Data Streams\. The results of this calculation are persisted to a DynamoDB table\.<a name="kinesis-gs-viewing-results-tables"></a>

**To view the Amazon DynamoDB tables**

1. Open the DynamoDB console at [https://console\.aws\.amazon\.com/dynamodb/](https://console.aws.amazon.com/dynamodb/)\.

1. On the navigation pane, select **Tables**\.

1. There are two tables created by the sample application:
   + KinesisDataVisSampleApp\-KCLDynamoDBTable\-\[randomString\]—Manages state information\.
   + KinesisDataVisSampleApp\-CountsDynamoDBTable\-\[randomString\]—Persists the results of the Top\-N analysis over a sliding window\.

1. Select the KinesisDataVisSampleApp\-KCLDynamoDBTable\-\[randomString\] table\. There are two entries in the table, indicating the particular shard \(leaseKey\), position in the stream \(checkpoint\), and the application reading the data \(leaseOwner\)\.

1. Select the KinesisDataVisSampleApp\-CountsDynamoDBTable\-\[randomString\] table\. You can see the aggregated visitor counts \(referrerCounts\) that the data consumer calculates as part of the sliding window analysis\.

**Kinesis Client Library \(KCL\)**  
Consumer applications can use the [Kinesis Client Library](https://github.com/awslabs/amazon-kinesis-client) \(KCL\) to simplify parallel processing of the stream\. The KCL takes care of many of the complex tasks associated with distributed computing, such as load\-balancing across multiple instances, responding to instance failures, checkpointing processed records, and reacting to resharding\. The KCL enables you to focus on writing record processing logic\.

The data consumer provides the KCL with position of the stream that it wants to read from, in this case specifying the latest possible data from the beginning of the stream\. The library uses this to call `GetShardIterator` on behalf of the consumer\. The consumer component also provides the client library with what to do with the records that are processed using an important KCL interface called `IRecordProcessor`\. The KCL calls `GetRecords` on behalf of the consumer and then processes those records as specified by `IRecordProcessor`\.
+ [Rows 92\-98](https://github.com/awslabs/amazon-kinesis-data-visualization-sample/blob/master/src/main/java/com/amazonaws/services/kinesis/samples/datavis/HttpReferrerCounterApplication.java#L92-98) of the HttpReferrerCounterApplication sample code configure the KCL\. This sets up the library with its initial configuration, such as the setting the position of the stream in which to read data\.
+ [Rows 104\-108](https://github.com/awslabs/amazon-kinesis-data-visualization-sample/blob/master/src/main/java/com/amazonaws/services/kinesis/samples/datavis/HttpReferrerCounterApplication.java#L104-108) of the HttpReferrerCounterApplication sample code inform the KCL of what logic to use when processing records using an important client library component, `IRecordProcessor`\.
+ [Rows 186\-203](https://github.com/awslabs/amazon-kinesis-data-visualization-sample/blob/d7fbcb994caad606635414b01c4eeada15c04f0b/src/main/java/com/amazonaws/services/kinesis/samples/datavis/kcl/CountingRecordProcessor.java#L186-203) of the CountingRecordProcessor sample code include the counting logic for the Top\-N analysis using `IRecordProcessor`\.

## Step 3: Delete Sample Application<a name="kinesis-gs-deleting-sample-resources"></a>

The sample application creates two shards, which incur shard usage charges while the application runs\. To ensure that your AWS account does not continue to be billed, delete your AWS CloudFormation stack after you finish with the sample application\.<a name="kinesis-gs-deleting-sample-resources-procedure"></a>

**To delete application resources**

1. Open the AWS CloudFormation console at [https://console\.aws\.amazon\.com/cloudformation](https://console.aws.amazon.com/cloudformation/)\.

1. Select the stack\.

1. Choose **Actions**, **Delete Stack**

1. When prompted for confirmation, choose **Yes, Delete**\.

The status changes to `DELETE_IN_PROGRESS` while AWS CloudFormation cleans up the resources associated with the sample application\. When AWS CloudFormation is finished cleaning up the resources, it removes the stack from the list\.

## Step 4: Next Steps<a name="kinesis-gs-next-steps"></a>
+ You can explore the source code for the [Data Visualization Sample Application](https://github.com/awslabs/amazon-kinesis-data-visualization-sample) on GitHub\.
+ You can find more advanced material about using the Kinesis Data Streams API in the [Developing Producers Using the Amazon Kinesis Data Streams API with the AWS SDK for Java](developing-producers-with-sdk.md), [Developing Consumers Using the Kinesis Data Streams API with the AWS SDK for Java](developing-consumers-with-sdk.md), and [Creating and Managing Streams](working-with-streams.md)\.
+ You can find sample application in the [AWS SDK for Java](https://aws.amazon.com/developers/getting-started/java/) that uses the SDK to put and get data from Kinesis Data Streams\.