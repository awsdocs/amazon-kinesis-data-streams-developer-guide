# Setting Up for Amazon Kinesis Data Streams<a name="before-you-begin"></a>

Before you use Amazon Kinesis Data Streams for the first time, complete the following tasks\.

**Topics**
+ [Sign Up for AWS](#setting-up-sign-up-for-aws)
+ [Download Libraries and Tools](#setting-up-downloads)
+ [Configure Your Development Environment](#setting-up-requirements)

## Sign Up for AWS<a name="setting-up-sign-up-for-aws"></a>

When you sign up for Amazon Web Services \(AWS\), your AWS account is automatically signed up for all services in AWS, including Kinesis Data Streams\. You are charged only for the services that you use\.

If you have an AWS account already, skip to the next task\. If you don't have an AWS account, use the following procedure to create one\.

**To sign up for an AWS account**

1. Open [https://portal\.aws\.amazon\.com/billing/signup](https://portal.aws.amazon.com/billing/signup)\.

1. Follow the online instructions\.

   Part of the sign\-up procedure involves receiving a phone call and entering a verification code on the phone keypad\.

## Download Libraries and Tools<a name="setting-up-downloads"></a>

The following libraries and tools will help you work with Kinesis Data Streams: 
+ The [Amazon Kinesis API Reference](https://docs.aws.amazon.com/kinesis/latest/APIReference/) is the basic set of operations that Kinesis Data Streams supports\. For more information about performing basic operations using Java code, see the following:
  + [Developing Producers Using the Amazon Kinesis Data Streams API with the AWS SDK for Java](developing-producers-with-sdk.md)
  + [Developing Custom Consumers with Shared Throughput Using the AWS SDK for Java ](developing-consumers-with-sdk.md)
  + [Creating and Managing Streams](working-with-streams.md)
+ The AWS SDKs for [Go](https://docs.aws.amazon.com/sdk-for-go/api/service/kinesis/), [Java](https://aws.amazon.com/developers/getting-started/java/), [JavaScript](https://aws.amazon.com/sdkforbrowser/), [\.NET](https://aws.amazon.com/developers/getting-started/net/), [Node\.js](https://aws.amazon.com/developers/getting-started/nodejs/), [PHP](https://aws.amazon.com/developers/getting-started/php/), [Python](https://github.com/boto/boto), and [Ruby](https://aws.amazon.com/developers/getting-started/ruby/) include Kinesis Data Streams support and samples\. If your version of the AWS SDK for Java does not include samples for Kinesis Data Streams, you can also download them from [GitHub](https://github.com/aws/aws-sdk-java/tree/master/src/samples)\.
+ The Kinesis Client Library \(KCL\) provides an easy\-to\-use programming model for processing data\. The KCL can help you get started quickly with Kinesis Data Streams in Java, Node\.js, \.NET, Python, and Ruby\. For more information see [Reading Data from Streams](building-consumers.md)\.
+ The [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/) supports Kinesis Data Streams\. The AWS CLI enables you to control multiple AWS services from the command line and automate them through scripts\.

## Configure Your Development Environment<a name="setting-up-requirements"></a>

To use the KCL, ensure that your Java development environment meets the following requirements:
+ Java 1\.7 \(Java SE 7 JDK\) or later\. You can download the latest Java software from [Java SE Downloads](http://www.oracle.com/technetwork/java/javase/downloads/index.html) on the Oracle website\.
+ Apache Commons package \(Code, HTTP Client, and Logging\)
+ Jackson JSON processor

Note that the [AWS SDK for Java](https://aws.amazon.com/sdkforjava/) includes Apache Commons and Jackson in the third\-party folder\. However, the SDK for Java works with Java 1\.6, while the Kinesis Client Library requires Java 1\.7\.