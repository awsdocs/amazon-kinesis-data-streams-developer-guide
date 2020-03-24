# Step 7: Finishing Up<a name="tutorial-stock-data-kplkcl2-finish"></a>

Because you are paying to use the Kinesis data stream, make sure that you delete it and the corresponding Amazon DynamoDB table when you are done with it\. Nominal charges occur on an active stream even when you aren't sending and getting records\. This is because an active stream is using resources by continuously "listening" for incoming records and requests to get records\.

**To delete the stream and table**

1. Shut down any producers and consumers that you may still have running\.

1. Open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose the stream that you created for this application \(`StockTradeStream`\)\.

1. Choose **Delete Stream**\.

1. Open the DynamoDB console at [https://console\.aws\.amazon\.com/dynamodb/](https://console.aws.amazon.com/dynamodb/)\.

1. Delete the `StockTradesProcessor` table\.

## Summary<a name="tutorial-stock-data-kplkcl2-summary"></a>

Processing a large amount of data in near\-real time doesn’t require writing any magical code or developing a huge infrastructure\. It is as simple as writing logic to process a small amount of data \(like writing `processRecord(Record)`\) but using Kinesis Data Streams to scale so that it works for a large amount of streaming data\. You don’t have to worry about how your processing would scale because Kinesis Data Streams handles it for you\. All you have to do is send your streaming records to Kinesis Data Streams and write the logic to process each new record received\. 

Here are some potential enhancements for this application\.

**Aggregate across all shards**  
Currently, you get stats resulting from aggregation of the data records received by a single worker from a single shard\. \(A shard cannot be processed by more than one worker in a single application at the same time\.\) Of course, when you scale and have more than one shard, you might want to aggregate across all shards\. You can do this by having a pipeline architecture where the output of each worker is fed into another stream with a single shard, which is processed by a worker that aggregates the outputs of the first stage\. Because the data from the first stage is limited \(one sample per minute per shard\), it would easily be handled by one shard\.

**Scale processing**  
When the stream scales up to have many shards \(because many producers are sending data\), the way to scale the processing is to add more workers\. You can run the workers in Amazon EC2 instances and use Auto Scaling groups\.

**Use connectors to Amazon S3/DynamoDB/Amazon Redshift/Storm**  
As a stream is continuously processed, its output can be sent to other destinations\. AWS provides [connectors](https://github.com/awslabs/amazon-kinesis-connectors) to integrate Kinesis Data Streams with other AWS services and third\-party tools\.