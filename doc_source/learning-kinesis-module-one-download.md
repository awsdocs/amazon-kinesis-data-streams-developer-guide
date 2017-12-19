# Step 3: Download and Build Implementation Code<a name="learning-kinesis-module-one-download"></a>

Skeleton code has been provided for you, containing a stub implementation for both the stock trade stream ingestion \(*producer*\) as well as the processing of the data \(*consumer*\)\. The following procedure shows how to complete the implementations\. 

**To download and build the implementation code**

1. Download the [source code](https://github.com/awslabs/amazon-kinesis-learning/tree/learning-module-1) on to your computer\.

1. Create a project in your favorite IDE with the source code, adhering to the provided folder structure\.

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

If you complete these steps successfully, you are now ready to move to the next section\. If your build generates errors at any stage, you need to investigate and fix them before proceeding\.