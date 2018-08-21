# Step 3: Download and Build the Implementation Code<a name="learning-kinesis-module-one-download"></a>

Skeleton code is provided for the [Tutorial: Analyzing Real\-Time Stock Data Using Kinesis Data Streams](learning-kinesis-module-one.md)\. It contains a stub implementation for both the stock trade stream ingestion \(*producer*\) and the processing of the data \(*consumer*\)\. The following procedure shows how to complete the implementations\. 

**To download and build the implementation code**

1. Download the [source code](https://github.com/awslabs/amazon-kinesis-learning/tree/learning-module-1) to your computer\.

1. Create a project in your favorite IDE with the source code, adhering to the provided directory structure\.

1. Add the following libraries to the project:
   + Amazon Kinesis Client Library \(KCL\)
   + AWS SDK
   + Apache HttpCore
   + Apache HttpClient
   + Apache Commons Lang
   + Apache Commons Logging
   + Guava \(Google Core Libraries For Java\)
   + Jackson Annotations
   + Jackson Core
   + Jackson Databind
   + Jackson Dataformat: CBOR
   + Joda Time

1. Depending on your IDE, the project might be built automatically\. If not, build the project using the appropriate steps for your IDE\.

If you complete these steps successfully, you are now ready to move to the next section, [Step 4: Implement the Producer](learning-kinesis-module-one-producer.md)\. If your build generates errors at any stage, investigate and fix them before proceeding\.

## Next Steps<a name="learning-kinesis-module-one-download-next"></a>

[[Step 4: Implement the Producer](learning-kinesis-module-one-producer.md)Step 4: Implement the Producer](learning-kinesis-module-one-producer.md)