# Part One: Streams, Producers, and Consumers<a name="learning-kinesis-module-one"></a>

This is Part One of a multi\-part series to learn how to develop for Amazon Kinesis Data Streams in Java\. If you are unsure which streaming service is right for you, see the comparison information at [What is Streaming Data?](https://aws.amazon.com/streaming-data/)\.

The scenario for this module is to ingest stock trades into a stream and write a simple application that performs calculations on the stream\. In Part One, you learn how to send a stream of records to Kinesis Data Streams and implement an application that consumes and processes the records in near real time\. In subsequent parts of this series, the scenario is extended to include more intermediate and advanced design and programming considerations for the stock trade analysis model that apply to most Kinesis Data Streams business applications\.

**Important**  
After you create a stream, your account incurs nominal charges for Kinesis Data Streams usage because Kinesis Data Streams is not eligible for the AWS Free Tier\. After the consumer application starts, it also incurs nominal charges for DynamoDB usage\. DynamoDB is used by the consumer application to track processing state\. When you are finished with this application, delete your AWS resources to stop incurring charges\. For more information, see [Step 7: Finishing Up](learning-kinesis-module-one-finish.md)\.

The code does not access actual stock market data, but instead simulates the stream of stock trades\. It does so by using a random stock trade generator that has a starting point of real market data for the top 25 stocks by market capitalization as of February 2015\. If you have access to a real time stream of stock trades, you might be interested in deriving useful, timely statistics from that stream\. For example, you might want to perform a sliding window analysis where you determine the most popular stock purchased in the last 5 minutes\. Or you might want a notification whenever there is a sell order that is too large \(that is, it has too many shares\)\. The code shown in this series can be extended to provide such functionality\.

You can work through the steps in this module on your desktop or laptop and run both the producer and consumer code on the same machine or any platform that supports the defined requirements, such as Amazon EC2\.

The examples shown use the US West \(Oregon\) Region, but they work on any of [the Regions that support Kinesis Data Streams](http://docs.aws.amazon.com/general/latest/gr/rande.html#ak_region)\.

**Topics**
+ [Prerequisites](learning-kinesis-module-one-begin.md)
+ [Step 1: Create a Stream](learning-kinesis-module-one-create-stream.md)
+ [Step 2: Create IAM Policy and User](learning-kinesis-module-one-iam.md)
+ [Step 3: Download and Build Implementation Code](learning-kinesis-module-one-download.md)
+ [Step 4: Implement the Producer](learning-kinesis-module-one-producer.md)
+ [Step 5: Implement the Consumer](learning-kinesis-module-one-consumer.md)
+ [Step 6: \(Optional\) Extending the Consumer](learning-kinesis-module-one-consumer-extension.md)
+ [Step 7: Finishing Up](learning-kinesis-module-one-finish.md)