# Permissions to Use User\-Generated KMS Master Keys<a name="permissions-user-key-KMS"></a>

Before you can use server\-side encryption with a user\-generated KMS master key, you must configure AWS KMS key policies to allow encryption of streams and encryption and decryption of stream records\. For examples and more information about AWS KMS permissions, see [AWS KMS API Permissions: Actions and Resources Reference](https://docs.aws.amazon.com/kms/latest/developerguide/kms-api-permissions-reference.html)\. 

**Note**  
The use of the default service key for encryption does not require application of custom IAM permissions\.

Before you use user\-generated KMS master keys, ensure that your Kinesis stream producers and consumers \(IAM principals\) are users in the KMS master key policy\. Otherwise, writes and reads from a stream will fail, which could ultimately result in data loss, delayed processing, or hung applications\. You can manage permissions for KMS keys using IAM policies\. For more information, see [Using IAM Policies with AWS KMS](http://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html)\.

## Example Producer Permissions<a name="example-producer-permissions"></a>

Your Kinesis stream producers must have the `kms:GenerateDataKey` permission\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
        "Effect": "Allow",
        "Action": [
            "kms:GenerateDataKey"
        ],
        "Resource": "arn:aws:kms:us-west-2:123456789012:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }, 
    {
        "Effect": "Allow",
        "Action": [
            "kinesis:PutRecord",
            "kinesis:PutRecords"
        ],
        "Resource": "arn:aws:kinesis:*:123456789012:MyStream"
    }
  ]
}
```

## Example Consumer Permissions<a name="example-consumer-permissions"></a>

Your Kinesis stream consumers must have the `kms:Decrypt` permission\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
        "Effect": "Allow",
        "Action": [
            "kms:Decrypt"
        ],
        "Resource": "arn:aws:kms:us-west-2:123456789012:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }, 
    {
        "Effect": "Allow",
        "Action": [
            "kinesis:GetRecords",
            "kinesis:DescribeStream"
        ],
        "Resource": "arn:aws:kinesis:*:123456789012:MyStream"
    }
  ]
}
```

Amazon Kinesis Data Analytics and AWS Lambda use roles to consume Kinesis streams\. Make sure to add the `kms:Decrypt` permission to the roles that these consumers use\.

## Stream Administrator Permissions<a name="stream-administrator-permissions"></a>

Kinesis stream administrators must have authorization to call `kms:List*` and ```kms:DescribeKey*`\.