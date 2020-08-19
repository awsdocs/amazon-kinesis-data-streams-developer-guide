# Listing Shards<a name="kinesis-using-sdk-java-list-shards"></a>

A data stream can have one or more shards\. There are two methods for listing \(or retrieving\) shards from a data stream\.

**Topics**
+ [ListShards API \- Recommended](#kinesis-using-sdk-java-list-shards-method1)
+ [DescribeStream API \- Deprecated](#kinesis-using-sdk-java-retrieve-shards)

## ListShards API \- Recommended<a name="kinesis-using-sdk-java-list-shards-method1"></a>

The recommended method for listing or retrieving the shards from a data stream is to use the [ListShards](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListShards.html) API\. The following example shows how you can get a list of the shards in a data stream\. For a full description of the main operation used in this example and all of the parameters you can set for the operation, see [ListShards](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListShards.html)\.

```
import software.amazon.awssdk.services.kinesis.KinesisAsyncClient;
import software.amazon.awssdk.services.kinesis.model.ListShardsRequest;
import software.amazon.awssdk.services.kinesis.model.ListShardsResponse;

import java.util.concurrent.TimeUnit;

public class ShardSample {

    public static void main(String[] args) {

        KinesisAsyncClient client = KinesisAsyncClient.builder().build();

        ListShardsRequest request = ListShardsRequest
                .builder().streamName("myFirstStream")
                .build();

        try {
            ListShardsResponse response = client.listShards(request).get(5000, TimeUnit.MILLISECONDS);
            System.out.println(response.toString());
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

To run the previous code example you can use a POM file like the following one\.

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>kinesis.data.streams.samples</groupId>
    <artifactId>shards</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>kinesis</artifactId>
            <version>2.0.0</version>
        </dependency>
    </dependencies>
</project>
```

With the `ListShards` API, you can use the [ShardFilter](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ShardFilter.html) parameter to filter out the response of the API\. You can only specify one filter at a time\. 

If you use the `ShardFilter` parameter when invoking the ListShards API, the `Type` is the required property and must be specified\. If you specify the `AT_TRIM_HORIZON`, `FROM_TRIM_HORIZON`, or `AT_LATEST` types, you do not need to specify either the `ShardId` or the `Timestamp` optional properties\.

If you specify the `AFTER_SHARD_ID` type, you must also provide the value for the optional `ShardId` property\. The `ShardId` property is identical in fuctionality to the `ExclusiveStartShardId` parameter of the ListShards API\. When `ShardId` property is specified, the response includes the shards starting with the shard whose ID immediately follows the `ShardId` that you provided\.

If you specify the `AT_TIMESTAMP` or `FROM_TIMESTAMP_ID` type, you must also provide the value for the optional `Timestamp` property\. If you specify the `AT_TIMESTAMP` type, then all shards that were open at the provided timestamp are returned\. If you specify the `FROM_TIMESTAMP` type, then all shards starting from the provided timestamp to TIP are returned\. 

## DescribeStream API \- Deprecated<a name="kinesis-using-sdk-java-retrieve-shards"></a>

**Important**  
The information below descibes a currently deprecated way of retrieving shards from a data stream via the DescribeStream API\. It is currently highly recommended that you use the `ListShards` API to retrieve the shards that comprise the data stream\.

The response object returned by the `describeStream` method enables you to retrieve information about the shards that comprise the stream\. To retrieve the shards, call the `getShards` method on this object\. This method might not return all the shards from the stream in a single call\. In the following code, we check the `getHasMoreShards` method on `getStreamDescription` to see if there are additional shards that were not returned\. If so, that is, if this method returns `true`, we continue to call `getShards` in a loop, adding each new batch of returned shards to our list of shards\. The loop exits when `getHasMoreShards` returns `false`; that is, all shards have been returned\. Note that `getShards` does not return shards that are in the `EXPIRED` state\. For more information about shard states, including the `EXPIRED` state, see [Data Routing, Data Persistence, and Shard State after a Reshard](kinesis-using-sdk-java-after-resharding.md#kinesis-using-sdk-java-resharding-data-routing)\. 

```
DescribeStreamRequest describeStreamRequest = new DescribeStreamRequest();
describeStreamRequest.setStreamName( myStreamName );
List<Shard> shards = new ArrayList<>();
String exclusiveStartShardId = null;
do {
    describeStreamRequest.setExclusiveStartShardId( exclusiveStartShardId );
    DescribeStreamResult describeStreamResult = client.describeStream( describeStreamRequest );
    shards.addAll( describeStreamResult.getStreamDescription().getShards() );
    if (describeStreamResult.getStreamDescription().getHasMoreShards() && shards.size() > 0) {
        exclusiveStartShardId = shards.get(shards.size() - 1).getShardId();
    } else {
        exclusiveStartShardId = null;
    }
} while ( exclusiveStartShardId != null );
```