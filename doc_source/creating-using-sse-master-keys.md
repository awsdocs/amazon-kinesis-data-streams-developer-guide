# Creating and Using User\-Generated KMS Master Keys<a name="creating-using-sse-master-keys"></a>

This section describes how to create and use your own KMS master keys, instead of using the master key administered by Amazon Kinesis\.

## Creating User\-Generated KMS Master Keys<a name="creating-sse-master-keys"></a>

For instructions on creating your own master keys, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\. After you create keys for your account, the Kinesis Data Streams service returns these keys in the **KMS master key** list\.

## Using User\-Generated KMS Master Keys<a name="using-sse-master-keys"></a>

After the correct permissions are applied to your consumers, producers, and administrators, you can use custom KMS master keys in your own AWS account or another AWS account\. All KMS master keys in your account appear in the **KMS Master Key** list within the AWS Management Console\.

To use custom KMS master keys located in another account, you need permissions to use those keys\. You must also specify the ARN of the KMS master key in the ARN input box in the AWS Management Console\.