# Developing Enhanced Fan\-Out Consumers with the Kinesis Data Streams API<a name="building-enhanced-consumers-api"></a>

*Enhanced fan\-out* is an Amazon Kinesis Data Streams feature that enables consumers to receive records from a data stream with dedicated throughput of up to 2 MB of data per second per shard\. A consumer that uses enhanced fan\-out doesn't have to contend with other consumers that are receiving data from the stream\. For more information, see [Developing Custom Consumers with Dedicated Throughput \(Enhanced Fan\-Out\)](enhanced-consumers.md)\.

You can use API operations to build a consumer that uses enhanced fan\-out in Kinesis Data Streams\.

**To register a consumer with enhanced fan\-out using the Kinesis Data Streams API**

1. Call [RegisterStreamConsumer](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_RegisterStreamConsumer.html) to register your application as a consumer that uses enhanced fan\-out\. Kinesis Data Streams generates an Amazon Resource Name \(ARN\) for the consumer and returns it in the response\.

1. To start listening to a specific shard, pass the consumer ARN in a call to [SubscribeToShard](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_SubscribeToShard.html)\. Kinesis Data Streams then starts pushing the records from that shard to you, in the form of events of type [SubscribeToShardEvent](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_SubscribeToShardEvent.html) over an HTTP/2 connection\. The connection remains open for up to 5 minutes\. Call [SubscribeToShard](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_SubscribeToShard.html) again if you want to continue receiving records from the shard after the `future` that is returned by the call to [SubscribeToShard](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_SubscribeToShard.html) completes normally or exceptionally\.
**Note**  
`SubscribeToShard` API also returns the list of the child shards of the current shard when the end of the current shard is reached\. 

1. To deregister a consumer that is using enhanced fan\-out, call [DeregisterStreamConsumer](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_DeregisterStreamConsumer.html)\.

The following code is an example of how you can subscribe your consumer to a shard, renew the subscription periodically, and handle the events\.

```
    import software.amazon.awssdk.services.kinesis.KinesisAsyncClient;
    import software.amazon.awssdk.services.kinesis.model.ShardIteratorType;
    import software.amazon.awssdk.services.kinesis.model.SubscribeToShardEvent;
    import software.amazon.awssdk.services.kinesis.model.SubscribeToShardRequest;
    import software.amazon.awssdk.services.kinesis.model.SubscribeToShardResponseHandler;
     
    import java.util.concurrent.CompletableFuture;
     
    /**
     * See https://github.com/awsdocs/aws-doc-sdk-examples/blob/master/javav2/example_code/kinesis/src/main/java/com/example/kinesis/KinesisStreamEx.java
     * for complete code and more examples.
     */
    public class SubscribeToShardSimpleImpl {
     
        private static final String CONSUMER_ARN = "arn:aws:kinesis:us-east-1:123456789123:stream/foobar/consumer/test-consumer:1525898737";
        private static final String SHARD_ID = "shardId-000000000000";
     
        public static void main(String[] args) {
     
            KinesisAsyncClient client = KinesisAsyncClient.create();
     
            SubscribeToShardRequest request = SubscribeToShardRequest.builder()
                    .consumerARN(CONSUMER_ARN)
                    .shardId(SHARD_ID)
                    .startingPosition(s -> s.type(ShardIteratorType.LATEST)).build();
     
            // Call SubscribeToShard iteratively to renew the subscription periodically.
            while(true) {
                // Wait for the CompletableFuture to complete normally or exceptionally.
                callSubscribeToShardWithVisitor(client, request).join();
            }
     
            // Close the connection before exiting.
            // client.close();
        }
     
     
        /**
         * Subscribes to the stream of events by implementing the SubscribeToShardResponseHandler.Visitor interface.
         */
        private static CompletableFuture<Void> callSubscribeToShardWithVisitor(KinesisAsyncClient client, SubscribeToShardRequest request) {
            SubscribeToShardResponseHandler.Visitor visitor = new SubscribeToShardResponseHandler.Visitor() {
                @Override
                public void visit(SubscribeToShardEvent event) {
                    System.out.println("Received subscribe to shard event " + event);
                }
            };
            SubscribeToShardResponseHandler responseHandler = SubscribeToShardResponseHandler
                    .builder()
                    .onError(t -> System.err.println("Error during stream - " + t.getMessage()))
                    .subscriber(visitor)
                    .build();
            return client.subscribeToShard(request, responseHandler);
        }
    }
```

 If `event.ContinuationSequenceNumber` returns `null`, it indicates that a shard split or merge has occurred that involved this shard\. This shard is now in a `CLOSED` state, and you have read all available data records from this shard\. In this scenario, per example above, you can use `event.childShards` to learn about the new child shards of the shard that is being processed that were created by the split or merge\. For more information, see [ChildShard](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ChildShard.html)\.