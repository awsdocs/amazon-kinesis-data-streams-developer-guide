# Writing to your Kinesis Data Stream Using the KPL<a name="kinesis-kpl-writing"></a>

The following sections show sample code in a progression from the simplest possible "bare\-bones" producer on through to fully asynchronous code\.

## Barebones Producer Code<a name="kinesis-kpl-writing-code"></a>

The following code is all that is needed to write a minimal working producer\. The Kinesis Producer Library \(KPL\) user records are processed in the background\.

```
// KinesisProducer gets credentials automatically like 
// DefaultAWSCredentialsProviderChain. 
// It also gets region automatically from the EC2 metadata service. 
KinesisProducer kinesis = new KinesisProducer();  
// Put some records 
for (int i = 0; i < 100; ++i) {
    ByteBuffer data = ByteBuffer.wrap("myData".getBytes("UTF-8"));
    // doesn't block       
    kinesis.addUserRecord("myStream", "myPartitionKey", data); 
}  
// Do other stuff ...
```

## Responding to Results Synchronously<a name="kinesis-kpl-writing-synchronous"></a>

In the previous example, the code didn't check whether the KPL user records succeeded\. The KPL performs any retries needed to account for failures\. But if you want to check on the results, you can examine them using the `Future` objects that are returned from `addUserRecord`, as in the following example \(previous example shown for context\):

```
KinesisProducer kinesis = new KinesisProducer();  

// Put some records and save the Futures 
List<Future<UserRecordResult>> putFutures = new LinkedList<Future<UserRecordResult>>(); 
for (int i = 0; i < 100; i++) {
    ByteBuffer data = ByteBuffer.wrap("myData".getBytes("UTF-8"));
    // doesn't block 
    putFutures.add(
        kinesis.addUserRecord("myStream", "myPartitionKey", data)); 
}  

// Wait for puts to finish and check the results 
for (Future<UserRecordResult> f : putFutures) {
    UserRecordResult result = f.get(); // this does block     
    if (result.isSuccessful()) {         
        System.out.println("Put record into shard " + 
                            result.getShardId());     
    } else {
        for (Attempt attempt : result.getAttempts()) {
            // Analyze and respond to the failure         
        }
    }
}
```

## Responding to Results Asynchronously<a name="kinesis-kpl-writing-asynchronous"></a>

The previous example is calling `get()` on a `Future` object, which blocks execution\. If you don't want to block execution, you can use an asynchronous callback, as shown in the following example:

```
KinesisProducer kinesis = new KinesisProducer();

FutureCallback<UserRecordResult> myCallback = new FutureCallback<UserRecordResult>() {     
    @Override public void onFailure(Throwable t) {
        /* Analyze and respond to the failure  */ 
    };     
    @Override public void onSuccess(UserRecordResult result) { 
        /* Respond to the success */ 
    };
};

for (int i = 0; i < 100; ++i) {
    ByteBuffer data = ByteBuffer.wrap("myData".getBytes("UTF-8"));      
    ListenableFuture<UserRecordResult> f = kinesis.addUserRecord("myStream", "myPartitionKey", data);     
    // If the Future is complete by the time we call addCallback, the callback will be invoked immediately.
    Futures.addCallback(f, myCallback); 
}
```