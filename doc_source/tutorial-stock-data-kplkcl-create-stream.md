# Step 1: Create a Data Stream<a name="tutorial-stock-data-kplkcl-create-stream"></a>

In the first step of the [Tutorial: Process Real\-Time Stock Data Using KPL and KCL 1\.x[Tutorial: Process Real\-Time Stock Data Using KPL and KCL 1\.x](tutorial-stock-data-kplkcl.md)](tutorial-stock-data-kplkcl.md), you create the stream that you will use in subsequent steps\.

**To create a stream**

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation pane\.

1. In the navigation bar, expand the Region selector and choose a Region\.

1. Choose **Create Kinesis stream**\.

1. Enter a name for your stream \(for example, **StockTradeStream**\)\.

1. Enter **1** for the number of shards, but leave **Estimate the number of shards you'll need** collapsed\.

1. Choose **Create Kinesis stream**\.

On the **Kinesis streams** list page, the status of your stream is `CREATING` while the stream is being created\. When the stream is ready to use, the status changes to `ACTIVE`\. Choose the name of your stream\. In the page that appears, the **Details** tab displays a summary of your stream configuration\. The **Monitoring** section displays monitoring information for the stream\.

## Additional Information About Shards<a name="tutorial-stock-data-kplkcl-create-stream-info"></a>

When you begin to use Kinesis Data Streams outside of this tutorial, you might need to plan the stream creation process more carefully\. You should plan for expected maximum demand when you provision shards\. Using this scenario as an example, US stock market trading traffic peaks during the day \(Eastern time\) and demand estimates should be sampled from that time of day\. You then have a choice to provision for the maximum expected demand, or scale your stream up and down in response to demand fluctuations\. 

A *shard* is a unit of throughput capacity\. On the **Create Kinesis stream** page, expand **Estimate the number of shards you'll need**\. Enter the average record size, the maximum records written per second, and the number of consuming applications, using the following guidelines:

**Average record size**  
An estimate of the calculated average size of your records\. If you don't know this value, use the estimated maximum record size for this value\.

**Max records written**  
Take into account the number of entities providing data and the approximate number of records per second produced by each\. For example, if you are getting stock trade data from 20 trading servers and each generates 250 trades per second, the total number of trades \(records\) per second is 5000/second\. 

**Number of consuming applications**  
The number of applications that independently read from the stream to process the stream in a different way and produce different output\. Each application can have multiple instances running on different machines \(that is, run in a cluster\) so that it can keep up with a high volume stream\.

If the estimated number of shards shown exceeds your current shard limit, you might need to submit a request to increase that limit before you can create a stream with that number of shards\. To request an increase to your shard limit, use the [Kinesis Data Streams Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-kinesis)\. For more information about streams and shards, see [Creating and Updating Data Streams](amazon-kinesis-streams.md) and [Creating and Managing Streams](working-with-streams.md)\.

## Next Steps<a name="tutorial-stock-data-kplkcl-create-stream-next"></a>

[Step 2: Create an IAM Policy and User](tutorial-stock-data-kplkcl-iam.md)