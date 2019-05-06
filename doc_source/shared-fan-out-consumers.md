# Developing Amazon Kinesis Data Streams Consumers<a name="shared-fan-out-consumers"></a>

If you don't need dedicated throughput when receiving data from Kinesis Data Streams, and if you don't need read propagation delays under 200 ms, you can build consumer applications as described in the following topics\.

**Topics**
+ [Developing Consumers Using the Kinesis Client Library 1\.x](developing-consumers-with-kcl.md)
+ [Developing Consumers Using the Kinesis Client Library 2\.0](developing-consumers-with-kcl-v2.md)
+ [Developing Consumers Using the Kinesis Data Streams API with the AWS SDK for Java](developing-consumers-with-sdk.md)

For information about building consumers that can receive records from Kinesis data streams with dedicated throughput, see [Using Consumers with Enhanced Fan\-Out ](introduction-to-enhanced-consumers.md)\.