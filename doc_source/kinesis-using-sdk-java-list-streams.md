# Listing Streams<a name="kinesis-using-sdk-java-list-streams"></a>

As described in the previous section, streams are scoped to the AWS account associated with the AWS credentials used to instantiate the Kinesis Data Streams client and also to the Region specified for the client\. An AWS account could have many streams active at one time\. You can list your streams in the Kinesis Data Streams console, or programmatically\. The code in this section shows how to list all the streams for your AWS account\. 

```
ListStreamsRequest listStreamsRequest = new ListStreamsRequest();
listStreamsRequest.setLimit(20); 
ListStreamsResult listStreamsResult = client.listStreams(listStreamsRequest);
List<String> streamNames = listStreamsResult.getStreamNames();
```

This code example first creates a new instance of `ListStreamsRequest` and calls its `setLimit` method to specify that a maximum of 20 streams should be returned for each call to `listStreams`\. If you do not specify a value for `setLimit`, Kinesis Data Streams returns a number of streams less than or equal to the number in the account\. The code then passes `listStreamsRequest` to the `listStreams` method of the client\. The return value `listStreams` is stored in a `ListStreamsResult` object\. The code calls the `getStreamNames` method on this object and stores the returned stream names in the `streamNames` list\. Note that Kinesis Data Streams might return fewer streams than specified by the specified limit even if there are more streams than that in the account and Region\. To ensure that you retrieve all the streams, use the `getHasMoreStreams` method as described in the next code example\. 

```
while (listStreamsResult.getHasMoreStreams()) 
{
    if (streamNames.size() > 0) {
      listStreamsRequest.setExclusiveStartStreamName(streamNames.get(streamNames.size() - 1));
    }
    listStreamsResult = client.listStreams(listStreamsRequest);
    streamNames.addAll(listStreamsResult.getStreamNames());
}
```

This code calls the `getHasMoreStreams` method on `listStreamsRequest` to check if there are additional streams available beyond the ones returned in the initial call to `listStreams`\. If so, the code calls the `setExclusiveStartStreamName` method with the name of the last stream that was returned in the previous call to `listStreams`\. The `setExclusiveStartStreamName` method causes the next call to `listStreams` to start after that stream\. The group of stream names returned by that call is then added to the `streamNames` list\. This process continues until all the stream names have been collected in the list\.

 The streams returned by `listStreams` can be in one of the following states: 
+ `CREATING`
+ `ACTIVE`
+ `UPDATING`
+ `DELETING`

You can check the state of a stream using the `describeStream` method, as shown in the previous section, [Creating a Stream](kinesis-using-sdk-java-create-stream.md)\.