# Getting Started with Amazon Kinesis Data Streams<a name="getting-started"></a>

The information in this section helps you get started using Amazon Kinesis Data Streams\. If you are new to Kinesis Data Streams, start by becoming familiar with the concepts and terminology presented in [Amazon Kinesis Data Streams Terminology and Concepts](key-concepts.md)\.

This section shows you how to perform basic Amazon Kinesis Data Streams operations using the AWS Command Line Interface\. You will learn fundamental Kinesis Data Streams data flow principles and the steps necessary to put and get data from an Kinesis data stream\.

**Topics**
+ [Install and Configure the AWS CLI](kinesis-tutorial-cli-installation.md)
+ [Perform Basic Kinesis Data Stream Operations Using the AWS CLI](fundamental-stream.md)

For CLI access, you need an access key ID and secret access key\. Use IAM user access keys instead of AWS account root user access keys\. IAM lets you securely control access to AWS services and resources in your AWS account\. For more information about creating access keys, see [Understanding and Getting Your Security Credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) in the *AWS General Reference*\.

You can find detailed step\-by\-step IAM and security key set up instructions at [Create an IAM User](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-an-iam-user)\.

In this section, the specific commands discussed are given verbatim, except where specific values are necessarily different for each run\. Also, the examples are using the US West \(Oregon\) region, but the steps in this section work in any of [the regions where Kinesis Data Streams is supported](https://docs.aws.amazon.com/general/latest/gr/rande.html#ak_region)\.