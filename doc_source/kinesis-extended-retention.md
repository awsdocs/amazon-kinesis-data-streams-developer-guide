# Changing the Data Retention Period<a name="kinesis-extended-retention"></a>

Amazon Kinesis Data Streams supports changes to the data record retention period of your stream\. A Kinesis data stream is an ordered sequence of data records meant to be written to and read from in real time\. Data records are therefore stored in shards in your stream temporarily\. The time period from when a record is added to when it is no longer accessible is called the *retention period*\. A Kinesis data stream stores records from 24 hours by default, up to 168 hours\. 

You can increase the retention period up to 168 hours using the [IncreaseStreamRetentionPeriod](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_IncreaseStreamRetentionPeriod.html) operation\. You can decrease the retention period down to a minimum of 24 hours using the [DecreaseStreamRetentionPeriod](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_DecreaseStreamRetentionPeriod.html) operation\. The request syntax for both operations includes the stream name and the retention period in hours\. Finally, you can check the current retention period of a stream by calling the [DescribeStream](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeStream.html) operation\.

Both operations are easy to use\. The following is an example of changing the retention period using the AWS CLI:

```
aws kinesis increase-stream-retention-period --stream-name retentionPeriodDemo --retention-period-hours 72
```

Kinesis Data Streams stops making records inaccessible at the old retention period within several minutes of increasing the retention period\. For example, changing the retention period from 24 hours to 48 hours means that records added to the stream 23 hours 55 minutes prior are still available after 24 hours\.

Kinesis Data Streams almost immediately makes records older than the new retention period inaccessible upon decreasing the retention period\. Therefore, take great care when calling the [DecreaseStreamRetentionPeriod](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_DecreaseStreamRetentionPeriod.html) operation\.

Set your data retention period to ensure that your consumers are able to read data before it expires, if problems occur\. You should carefully consider all possibilities, such as an issue with your record processing logic or a downstream dependency being down for a long period of time\. Think of the retention period as a safety net to allow more time for your data consumers to recover\. The retention period API operations allow you to set this up proactively or to respond to operational events reactively\.

 Additional charges apply for streams with a retention period set above 24 hours\. For more information, see [Amazon Kinesis Data Streams Pricing](https://aws.amazon.com/kinesis/data-streams/pricing/)\.