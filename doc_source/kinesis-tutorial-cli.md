# Tutorial: Getting Started With Amazon Kinesis Data Streams Using AWS CLI<a name="kinesis-tutorial-cli"></a>

This tutorial shows you how to perform basic Amazon Kinesis Data Streams operations using the AWS Command Line Interface\. You will learn fundamental Kinesis Data Streams data flow principles and the steps necessary to put and get data from an Kinesis data stream\.

 For CLI access, you need an access key ID and secret access key\. Use IAM user access keys instead of AWS account root user access keys\. IAM lets you securely control access to AWS services and resources in your AWS account\. For more information about creating access keys, see [Understanding and Getting Your Security Credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) in the *AWS General Reference*\. 

You can find detailed step\-by\-step IAM and security key set up instructions at [Create an IAM User](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-an-iam-user)\.

In this tutorial, the specific commands discussed will be given verbatim, except where specific values will necessarily be different for each run\. Also, the examples are using the US West \(Oregon\) region, but this tutorial will work on any of [the regions that support Kinesis Data Streams](https://docs.aws.amazon.com/general/latest/gr/rande.html#ak_region)\.

**Topics**
+ [Install and Configure the AWS CLI](kinesis-tutorial-cli-installation.md)
+ [Perform Basic Stream Operations](fundamental-stream.md)