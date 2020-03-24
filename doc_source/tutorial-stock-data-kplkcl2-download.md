# Step 3: Download and Build the Code<a name="tutorial-stock-data-kplkcl2-download"></a>

This topic provides sample implementation code for the sample stock trades ingestion into the data stream \(*producer*\) and the processing of this data \(*consumer*\)\.

**To download and build the code**

1. Download the source code from the [https://github\.com/aws\-samples/amazon\-kinesis\-learning](https://github.com/aws-samples/amazon-kinesis-learning) GitHub repo to your computer\.

1. Create a project in your IDE with the source code, adhering to the provided directory structure\.

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

If you complete these steps successfully, you are now ready to move to the next section, [Step 4: Implement the Producer](tutorial-stock-data-kplkcl2-producer.md)\. 

## Next Steps<a name="tutorial-stock-data-kplkcl2-download-next"></a>

[[Step 4: Implement the Producer](tutorial-stock-data-kplkcl2-producer.md)Step 4: Implement the Producer](tutorial-stock-data-kplkcl2-producer.md)