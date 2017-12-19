# Logging Amazon Kinesis Streams API Calls Using AWS CloudTrail<a name="logging-using-cloudtrail"></a>

Amazon Kinesis Streams is integrated with AWS CloudTrail, which captures API calls made by or on behalf of Kinesis Streams and delivers the log files to the Amazon S3 bucket that you specify\. The API calls can be made indirectly by using the Kinesis Streams console or directly by using the Kinesis Streams API\. Using the information collected by CloudTrail, you can determine what request was made to Kinesis Streams, the source IP address from which the request was made, who made the request, when it was made, and so on\. To learn more about CloudTrail, including how to configure and enable it, see the *[AWS CloudTrail User Guide](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)*\. 

## Kinesis Streams and CloudTrail<a name="kinesis-info-in-cloudtrail"></a>

CloudTrail logging is enabled by default\. Calls made to Kinesis Streams actions are tracked in log files\. Records for Kinesis Streams are written in a log file, together with records from any other AWS service enabled for CloudTrail logging\. CloudTrail determines when to create and write to a new file based on the specified time period and file size\.

The following actions are supported:

+ [AddTagsToStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_AddTagsToStream.html)

+ [CreateStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_CreateStream.html)

+ [DecreaseStreamRetentionPeriod](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DecreaseStreamRetentionPeriod.html)

+ [DeleteStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DeleteStream.html)

+ [DescribeLimits](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeLimits.html)

+ [DescribeStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeStream.html)

+ [DescribeStreamSummary](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DescribeStreamSummary.html)

+ [DisableEnhancedMonitoring](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_DisableEnhancedMonitoring.html)

+ [EnableEnhancedMonitoring](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_EnableEnhancedMonitoring.html)

+ [IncreaseStreamRetentionPeriod](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_IncreaseStreamRetentionPeriod.html)

+ [ListStreams](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListStreams.html)

+ [ListTagsForStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListTagsForStream.html)

+ [MergeShards](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_MergeShards.html)

+ [RemoveTagsFromStream](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_RemoveTagsFromStream.html)

+ [SplitShard](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_SplitShard.html)

+ [StartStreamEncryption](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_StartStreamEncryption.html)

+ [StopStreamEncryption](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_StopStreamEncryption.html)

+ [UpdateShardCount](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_UpdateShardCount.html)

Each log entry contains information about who generated the request\. For example, if a request is made to create a stream \(CreateStream\), the user identity of the person or service that made the request is logged\. The user identity information helps you determine whether the request was made with root or IAM user credentials, with temporary security credentials for a role or federated user, or by another AWS service\. For more information, see the [userIdentity Element](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/event_reference_user_identity.html) in the *AWS CloudTrail User Guide*\.

You can store your log files in your bucket for as long as you need to, but you can also define Amazon S3 lifecycle rules to archive or delete log files automatically\. By default, your log files are encrypted by using Amazon S3 server\-side encryption \(SSE\)\.

You can also aggregate Kinesis Streams log files from multiple AWS regions and multiple AWS accounts into a single Amazon S3 bucket\. For information, see [Aggregating CloudTrail Log Files to a Single Amazon S3 Bucket](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/aggregating_logs_top_level.html) in the *AWS CloudTrail User Guide*\.

You can have CloudTrail publish SNS notifications when new log files are delivered if you want to take quick action upon log file delivery\. For information, see [Configuring Amazon SNS Notifications](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html) in the *AWS CloudTrail User Guide*\.

## Log File Entries for Kinesis Streams<a name="kinesis-log-entries"></a>

CloudTrail log files can contain one or more log entries, where each entry is made up of multiple JSON\-formatted events\. A log entry represents a single request from any source and includes information about the requested action, any parameters, the date and time of the action, and so on\. The log entries are not guaranteed to be in any particular order\. That is, this is not an ordered stack trace of API calls\.

The following is an example CloudTrail log entry\.

```
{
    "Records": [
        {
            "eventVersion": "1.01",
            "userIdentity": {
                "type": "IAMUser",
                "principalId": "EX_PRINCIPAL_ID",
                "arn": "arn:aws:iam::012345678910:user/Alice",
                "accountId": "012345678910",
                "accessKeyId": "EXAMPLE_KEY_ID",
                "userName": "Alice"
            },
            "eventTime": "2014-04-19T00:16:31Z",
            "eventSource": "kinesis.amazonaws.com",
            "eventName": "CreateStream",
            "awsRegion": "us-east-1",
            "sourceIPAddress": "127.0.0.1",
            "userAgent": "aws-sdk-java/unknown-version Linux/x.xx",
            "requestParameters": {
                "shardCount": 1,
                "streamName": "GoodStream"
            },
            "responseElements": null,
            "requestID": "db6c59f8-c757-11e3-bc3b-57923b443c1c",
            "eventID": "b7acfcd0-6ca9-4ee1-a3d7-c4e8d420d99b"
        },
        {
            "eventVersion": "1.01",
            "userIdentity": {
                "type": "IAMUser",
                "principalId": "EX_PRINCIPAL_ID",
                "arn": "arn:aws:iam::012345678910:user/Alice",
                "accountId": "012345678910",
                "accessKeyId": "EXAMPLE_KEY_ID",
                "userName": "Alice"
            },
            "eventTime": "2014-04-19T00:17:06Z",
            "eventSource": "kinesis.amazonaws.com",
            "eventName": "DescribeStream",
            "awsRegion": "us-east-1",
            "sourceIPAddress": "127.0.0.1",
            "userAgent": "aws-sdk-java/unknown-version Linux/x.xx",
            "requestParameters": {
                "streamName": "GoodStream"
            },
            "responseElements": null,
            "requestID": "f0944d86-c757-11e3-b4ae-25654b1d3136",
            "eventID": "0b2f1396-88af-4561-b16f-398f8eaea596"
        },
        {
            "eventVersion": "1.01",
            "userIdentity": {
                "type": "IAMUser",
                "principalId": "EX_PRINCIPAL_ID",
                "arn": "arn:aws:iam::012345678910:user/Alice",
                "accountId": "012345678910",
                "accessKeyId": "EXAMPLE_KEY_ID",
                "userName": "Alice"
            },
            "eventTime": "2014-04-19T00:15:02Z",
            "eventSource": "kinesis.amazonaws.com",
            "eventName": "ListStreams",
            "awsRegion": "us-east-1",
            "sourceIPAddress": "127.0.0.1",
            "userAgent": "aws-sdk-java/unknown-version Linux/x.xx",
            "requestParameters": {
                "limit": 10
            },
            "responseElements": null,
            "requestID": "a68541ca-c757-11e3-901b-cbcfe5b3677a",
            "eventID": "22a5fb8f-4e61-4bee-a8ad-3b72046b4c4d"
        },
        {
            "eventVersion": "1.01",
            "userIdentity": {
                "type": "IAMUser",
                "principalId": "EX_PRINCIPAL_ID",
                "arn": "arn:aws:iam::012345678910:user/Alice",
                "accountId": "012345678910",
                "accessKeyId": "EXAMPLE_KEY_ID",
                "userName": "Alice"
            },
            "eventTime": "2014-04-19T00:17:07Z",
            "eventSource": "kinesis.amazonaws.com",
            "eventName": "DeleteStream",
            "awsRegion": "us-east-1",
            "sourceIPAddress": "127.0.0.1",
            "userAgent": "aws-sdk-java/unknown-version Linux/x.xx",
            "requestParameters": {
                "streamName": "GoodStream"
            },
            "responseElements": null,
            "requestID": "f10cd97c-c757-11e3-901b-cbcfe5b3677a",
            "eventID": "607e7217-311a-4a08-a904-ec02944596dd"
        },
        {
            "eventVersion": "1.01",
            "userIdentity": {
                "type": "IAMUser",
                "principalId": "EX_PRINCIPAL_ID",
                "arn": "arn:aws:iam::012345678910:user/Alice",
                "accountId": "012345678910",
                "accessKeyId": "EXAMPLE_KEY_ID",
                "userName": "Alice"
            },
            "eventTime": "2014-04-19T00:15:03Z",
            "eventSource": "kinesis.amazonaws.com",
            "eventName": "SplitShard",
            "awsRegion": "us-east-1",
            "sourceIPAddress": "127.0.0.1",
            "userAgent": "aws-sdk-java/unknown-version Linux/x.xx",
            "requestParameters": {
                "shardToSplit": "shardId-000000000000",
                "streamName": "GoodStream",
                "newStartingHashKey": "11111111"
            },
            "responseElements": null,
            "requestID": "a6e6e9cd-c757-11e3-901b-cbcfe5b3677a",
            "eventID": "dcd2126f-c8d2-4186-b32a-192dd48d7e33"
        },
        {
            "eventVersion": "1.01",
            "userIdentity": {
                "type": "IAMUser",
                "principalId": "EX_PRINCIPAL_ID",
                "arn": "arn:aws:iam::012345678910:user/Alice",
                "accountId": "012345678910",
                "accessKeyId": "EXAMPLE_KEY_ID",
                "userName": "Alice"
            },
            "eventTime": "2014-04-19T00:16:56Z",
            "eventSource": "kinesis.amazonaws.com",
            "eventName": "MergeShards",
            "awsRegion": "us-east-1",
            "sourceIPAddress": "127.0.0.1",
            "userAgent": "aws-sdk-java/unknown-version Linux/x.xx",
            "requestParameters": {
                "streamName": "GoodStream",
                "adjacentShardToMerge": "shardId-000000000002",
                "shardToMerge": "shardId-000000000001"
            },
            "responseElements": null,
            "requestID": "e9f9c8eb-c757-11e3-bf1d-6948db3cd570",
            "eventID": "77cf0d06-ce90-42da-9576-71986fec411f"
        }
    ]
}
```