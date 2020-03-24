# Prerequisites<a name="tutorial-stock-data-kplkcl2-begin"></a>

You must meet the following requirements in order to complete this tutorial:

## Amazon Web Services Account<a name="tutorial-stock-data-kplkcl2-begin-aws"></a>

Before you begin, ensure that you are familiar with the concepts discussed in [Amazon Kinesis Data Streams Terminology and Concepts](key-concepts.md), particularly with streams, shards, producers, and consumers\. It is also helpful to have completed the steps in the following guide: [Install and Configure the AWS CLI](kinesis-tutorial-cli-installation.md)\.

You must have an AWS account and a web browser to access the AWS Management Console\.

For console access, use your IAM user name and password to sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) from the [IAM sign\-in page](https://docs.aws.amazon.com/IAM/latest/UserGuide/console.html)\. IAM lets you securely control access to AWS services and resources in your AWS account\. For details about console and programmatic credentials, see [Understanding and Getting Your Security Credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) in the *AWS General Reference*\.

For more information about IAM and security key setup instructions, see [Create an IAM User](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-an-iam-user)\.

## System Software Requirements<a name="tutorial-stock-data-kplkcl2-begin-sys"></a>

The system that you are using to run the application must have Java 7 or higher installed\. To download and install the latest Java Development Kit \(JDK\), go to [Oracle's Java SE installation site](http://www.oracle.com/technetwork/java/javase/downloads/index.html)\.

If you have a Java IDE, such as [Eclipse](https://www.eclipse.org/downloads/), you can use it to open, edit, build, and run the source code\.

You need the latest [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) version\. If you are using Eclipse as your IDE, you can install the [AWS Toolkit for Eclipse](https://aws.amazon.com/eclipse/) instead\. 

The consumer application requires the Kinesis Client Library \(KCL\) version 2\.2\.9 or higher, which you can obtain from GitHub at [https://github\.com/awslabs/amazon\-kinesis\-client/tree/master](https://github.com/awslabs/amazon-kinesis-client/tree/master)\.

## Next Steps<a name="tutorial-stock-data-kplkcl2-begin-next"></a>

[Step 1: Create a Data Stream](tutorial-stock-data-kplkcl2-create-stream.md)