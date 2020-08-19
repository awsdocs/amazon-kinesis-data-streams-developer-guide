# Step 2: Create an IAM Policy and User<a name="tutorial-stock-data-kplkcl-iam"></a>

Security best practices for AWS dictate the use of fine\-grained permissions to control access to different resources\. AWS Identity and Access Management \(IAM\) allows you to manage users and user permissions in AWS\. An [IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/PoliciesOverview.html) explicitly lists actions that are allowed and the resources on which the actions are applicable\.

The following are the minimum permissions generally required for a Kinesis Data Streams producer and consumer\.


**Producer**  

| Actions | Resource | Purpose | 
| --- | --- | --- | 
| DescribeStream, DescribeStreamSummary, DescribeStreamConsumer | Kinesis data stream | Before attempting to write records, the producer checks if the stream exists and is active, and if the shards are contained in the stream, and if the stream has a consumer\. | 
| SubscribeToShard, RegisterStreamConsumer | Kinesis data stream | Subscribes and register a consumers to a Kinesis Data Stream shard\. | 
| PutRecord, PutRecords | Kinesis data stream | Write records to Kinesis Data Streams\. | 


**Consumer**  

| **Actions** | **Resource** | **Purpose** | 
| --- | --- | --- | 
| DescribeStream | Kinesis data stream | Before attempting to read records, the consumer checks if the stream exists and is active, and if the shards are contained in the stream\. | 
| GetRecords, GetShardIterator  | Kinesis data stream | Read records from a Kinesis Data Streams shard\. | 
| CreateTable, DescribeTable, GetItem, PutItem, Scan, UpdateItem | Amazon DynamoDB table | If the consumer is developed using the Kinesis Client Library \(KCL\), it needs permissions to a DynamoDB table to track the processing state of the application\. The first consumer started creates the table\.  | 
| DeleteItem | Amazon DynamoDB table | For when the consumer performs split/merge operations on Kinesis Data Streams shards\. | 
| PutMetricData | Amazon CloudWatch log | The KCL also uploads metrics to CloudWatch, which are useful for monitoring the application\. | 

For this application, you create a single IAM policy that grants all of the preceding permissions\. In practice, you might want to consider creating two policies, one for producers and one for consumers\.

**To create an IAM policy**

1. Locate the Amazon Resource Name \(ARN\) for the new stream\. You can find this ARN listed as **Stream ARN** at the top of the **Details** tab\. The ARN format is as follows:

   ```
   arn:aws:kinesis:region:account:stream/name
   ```  
*region*  
The Region code; for example, `us-west-2`\. For more information, see [Region and Availability Zone Concepts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-regions-availability-zones)\.  
*account*  
The AWS account ID, as shown in [Account Settings](https://console.aws.amazon.com/billing/home?#/account)\.  
*name*  
The name of the stream from [Step 1: Create a Data Stream](tutorial-stock-data-kplkcl-create-stream.md), which is `StockTradeStream`\.

1. Determine the ARN for the DynamoDB table to be used by the consumer \(and created by the first consumer instance\)\. It must be in the following format:

   ```
   arn:aws:dynamodb:region:account:table/name
   ```

   The Region and account are from the same place as the previous step, but this time *name* is the name of the table created and used by the consumer application\. The KCL used by the consumer uses the application name as the table name\. Use `StockTradesProcessor`, which is the application name used later\.

1. In the IAM console, in **Policies** \([https://console\.aws\.amazon\.com/iam/home\#policies](https://console.aws.amazon.com/iam/home#policies)\), choose **Create policy**\. If this is the first time that you have worked with IAM policies, choose **Get Started**, **Create Policy**\.

1. Choose **Select** next to **Policy Generator**\.

1. Choose **Amazon Kinesis** as the AWS service\.

1. Select `DescribeStream`, `GetShardIterator`, `GetRecords`, `PutRecord`, and `PutRecords` as the allowed actions\.

1. Enter the ARN that you created in Step 1\.

1. Use **Add Statement** for each of the following:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/streams/latest/dev/tutorial-stock-data-kplkcl-iam.html)

   The asterisk \(`*`\) that is used when specifying an ARN is not required\. In this case, it's because there is no specific resource in CloudWatch on which the `PutMetricData` action is invoked\.

1. Choose **Next Step**\.

1. Change **Policy Name** to `StockTradeStreamPolicy`, review the code, and choose **Create Policy**\.

The resulting policy document should look something like the following:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt123",
      "Effect": "Allow",
      "Action": [
        "kinesis:DescribeStream",
        "kinesis:PutRecord",
        "kinesis:PutRecords",
        "kinesis:GetShardIterator",
        "kinesis:GetRecords",
        "kinesis:ListShards",
        "kinesis:DescribeStreamSummary",
        "kinesis:RegisterStreamConsumer"
      ],
      "Resource": [
        "arn:aws:kinesis:us-west-2:123:stream/StockTradeStream"
      ]
    },
    {
      "Sid": "Stmt234",
      "Effect": "Allow",
      "Action": [
        "kinesis:SubscribeToShard",
        "kinesis:DescribeStreamConsumer"
      ],
      "Resource": [
        "arn:aws:kinesis:us-west-2:123:stream/StockTradeStream/*"
      ]
    },
    {
      "Sid": "Stmt456",
      "Effect": "Allow",
      "Action": [
        "dynamodb:*"
      ],
      "Resource": [
        "arn:aws:dynamodb:us-west-2:123:table/StockTradesProcessor"
      ]
    },
    {
      "Sid": "Stmt789",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricData"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

**To create an IAM user**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. On the **Users** page, choose **Add user**\.

1. For **User name**, type `StockTradeStreamUser`\.

1. For **Access type**, choose **Programmatic access**, and then choose **Next: Permissions**\.

1. Choose **Attach existing policies directly**\.

1. Search by name for the policy that you created\. Select the box to the left of the policy name, and then choose **Next: Review**\.

1. Review the details and summary, and then choose **Create user**\.

1. Copy the **Access key ID**, and save it privately\. Under **Secret access key**, choose **Show**, and save that key privately also\.

1. Paste the access and secret keys to a local file in a safe place that only you can access\. For this application, create a file named ` ~/.aws/credentials` \(with strict permissions\)\. The file should be in the following format:

   ```
   [default]
   aws_access_key_id=access key
   aws_secret_access_key=secret access key
   ```

**To attach an IAM policy to a user**

1. In the IAM console, open [Policies](https://console.aws.amazon.com/iam/home?#policies) and choose **Policy Actions**\. 

1. Choose `StockTradeStreamPolicy` and **Attach**\.

1. Choose `StockTradeStreamUser` and **Attach Policy**\.

## Next Steps<a name="tutorial-stock-data-kplkcl-iam-next"></a>

[Step 3: Download and Build the Implementation Code](tutorial-stock-data-kplkcl-download.md)