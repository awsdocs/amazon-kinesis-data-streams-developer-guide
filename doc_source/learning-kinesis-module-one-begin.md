# Prerequisites<a name="learning-kinesis-module-one-begin"></a>

The following are requirements for completing the [Tutorial: Analyzing Real\-Time Stock Data Using Kinesis Data Streams[Tutorial: Analyzing Real\-Time Stock Data Using Kinesis Data Streams](learning-kinesis-module-one.md)](learning-kinesis-module-one.md)\.

## Amazon Web Services Account<a name="learning-kinesis-module-one-begin-aws"></a>

Before you begin, ensure that you are familiar with the concepts discussed in [Kinesis Data Streams Key Concepts](key-concepts.md) and [Tutorial: Visualizing Web Traffic Using Amazon Kinesis Data Streams](kinesis-sample-application.md), particularly streams, shards, producers, and consumers\. It is also helpful to have completed [Tutorial: Visualizing Web Traffic Using Amazon Kinesis Data Streams](kinesis-sample-application.md) and [Tutorial: Getting Started With Amazon Kinesis Data Streams Using AWS CLI](kinesis-tutorial-cli.md)\.

You need an AWS account and a web browser to access the AWS Management Console\.

 For console access, use your IAM user name and password to sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) using the [IAM sign\-in page](https://docs.aws.amazon.com/IAM/latest/UserGuide/console.html)\. IAM lets you securely control access to AWS services and resources in your AWS account\. For more information about creating access keys, see [How Do I Get Security Credentials?](https://docs.aws.amazon.com/general/latest/gr/getting-aws-sec-creds.html) in the *AWS General Reference*\. 

For more information about IAM and security key setup instructions, see [Create an IAM User](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-an-iam-user)\.

## System Software Requirements<a name="learning-kinesis-module-one-begin-sys"></a>

The system used to run the application must have Java 7 or higher installed\. To download and install the latest Java Development Kit \(JDK\), go to [Oracle's Java SE installation site](http://www.oracle.com/technetwork/java/javase/downloads/index.html)\.

If you have a Java IDE, such as [Eclipse](https://www.eclipse.org/downloads/), you can open the source code, edit, build, and run it\.

You need the latest [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) version\. If you are using Eclipse as your IDE, you can install the [AWS Toolkit for Eclipse](https://aws.amazon.com/eclipse/) instead\. 

The consumer application requires the Kinesis Client Library \(KCL\) version 1\.2\.1 or higher, which you can obtain from GitHub at [Kinesis Client Library \(Java\)](https://github.com/awslabs/amazon-kinesis-client)\.

## Next Steps<a name="learning-kinesis-module-one-begin-next"></a>

[Step 1: Create a Data Stream](learning-kinesis-module-one-create-stream.md)