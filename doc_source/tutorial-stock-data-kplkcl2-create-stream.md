# Step 1: Create a Data Stream<a name="tutorial-stock-data-kplkcl2-create-stream"></a>

First, you must create the data stream that you will use in subsequent steps of this tutorial\.

**To create a stream**

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. Choose **Data Streams** in the navigation pane\.

1. In the navigation bar, expand the region selector and choose a region\.

1. Choose **Create Kinesis stream**\.

1. Enter a name for your data stream \(for example, **StockTradeStream**\)\.

1. Enter **1** for the number of shards, but leave **Estimate the number of shards you'll need** collapsed\.

1. Choose **Create Kinesis stream**\.

On the **Kinesis streams** list page, the status of your stream appears as `CREATING` while the stream is being created\. When the stream is ready to use, the status changes to `ACTIVE`\. 

If you choose the name of your stream, in the page that appears, the **Details** tab displays a summary of your data stream configuration\. The **Monitoring** section displays monitoring information for the stream\.

## Next Steps<a name="tutorial-stock-data-kplkcl2-create-stream-next"></a>

[Step 2: Create an IAM Policy and User](tutorial-stock-data-kplkcl2-iam.md)