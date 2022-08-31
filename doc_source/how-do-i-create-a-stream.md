# Creating a Stream via the AWS Management Console<a name="how-do-i-create-a-stream"></a>

You can create a stream using the Kinesis Data Streams console, the Kinesis Data Streams API, or the AWS Command Line Interface \(AWS CLI\)\.

**To create a data stream using the console**

1. Sign in to the AWS Management Console and open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. In the navigation bar, expand the Region selector and choose a Region\.

1. Choose **Create data stream**\.

1. On the **Create Kinesis stream** page, enter a name for your data stream and then choose either the **On\-demand** or **Provisioned** capacity mode\. The **On\-demand** mode is selected by default\. For more information, see [Choosing the Data Stream Capacity Mode](how-do-i-size-a-stream.md)\.

   With the **On\-demand** mode, you can then choose **Create Kinesis stream** to create your data stream\. With the **Provisioned** mode, you must then specify the number of shards you need, and then choose **Create Kinesis stream**\.

   On the **Kinesis streams** page, your stream's **Status** is **Creating** while the stream is being created\. When the stream is ready to use, the **Status** changes to **Active**\.

1. Choose the name of your stream\. The **Stream Details** page displays a summary of your stream configuration, along with monitoring information\.

**To create a stream using the Kinesis Data Streams API**
+ For information about creating a stream using the Kinesis Data Streams API, see [Creating a Stream via the APIs](kinesis-using-sdk-java-create-stream.md)\.

**To create a stream using the AWS CLI**
+ For information about creating a stream using the AWS CLI, see the [create\-stream](https://docs.aws.amazon.com/cli/latest/reference/kinesis/create-stream.html) command\.