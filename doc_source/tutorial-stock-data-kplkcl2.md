# Tutorial: Process Real\-Time Stock Data Using KPL and KCL 2\.x<a name="tutorial-stock-data-kplkcl2"></a>

The scenario for this tutorial involves ingesting stock trades into a data stream and writing a simple Amazon Kinesis Data Streams application that performs calculations on the stream\. You will learn how to send a stream of records to Kinesis Data Streams and implement an application that consumes and processes the records in near\-real time\.

**Important**  
After you create a stream, your account incurs nominal charges for Kinesis Data Streams usage because Kinesis Data Streams is not eligible for the AWS Free Tier\. After the consumer application starts, it also incurs nominal charges for Amazon DynamoDB usage\. The consumer application uses DynamoDB to track processing state\. When you are finished with this application, delete your AWS resources to stop incurring charges\. For more information, see [Step 7: Finishing Up](tutorial-stock-data-kplkcl2-finish.md)\.

The code does not access actual stock market data, but instead simulates the stream of stock trades\. It does so by using a random stock trade generator that has a starting point of real market data for the top 25 stocks by market capitalization as of February 2015\. If you have access to a real\-time stream of stock trades, you might be interested in deriving useful, timely statistics from that stream\. For example, you might want to perform a sliding window analysis where you determine the most popular stock purchased in the last 5 minutes\. Or you might want a notification whenever there is a sell order that is too large \(that is, it has too many shares\)\. You can extend the code in this series to provide such functionality\.

You can work through the steps in this tutorial on your desktop or laptop computer and run both the producer and consumer code on the same machine or any platform that supports the defined requirements\.

The examples shown use the US West \(Oregon\) region, but they work on any of the [AWS regions](https://docs.aws.amazon.com/general/latest/gr/rande.html#ak_region) that support Kinesis Data Streams\.

**Topics**
+ [Prerequisites](tutorial-stock-data-kplkcl2-begin.md)
+ [Step 1: Create a Data Stream](tutorial-stock-data-kplkcl2-create-stream.md)
+ [Step 2: Create an IAM Policy and User](tutorial-stock-data-kplkcl2-iam.md)
+ [Step 3: Download and Build the Code](tutorial-stock-data-kplkcl2-download.md)
+ [Step 4: Implement the Producer](tutorial-stock-data-kplkcl2-producer.md)
+ [Step 5: Implement the Consumer](tutorial-stock-data-kplkcl2-consumer.md)
+ [Step 6: \(Optional\) Extending the Consumer](tutorial-stock-data-kplkcl2-consumer-extension.md)
+ [Step 7: Finishing Up](tutorial-stock-data-kplkcl2-finish.md)