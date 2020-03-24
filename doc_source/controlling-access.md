# Controlling Access to Amazon Kinesis Data Streams Resources Using IAM<a name="controlling-access"></a>

AWS Identity and Access Management \(IAM\) enables you to do the following:
+ Create users and groups under your AWS account
+ Assign unique security credentials to each user under your AWS account
+ Control each user's permissions to perform tasks using AWS resources
+ Allow the users in another AWS account to share your AWS resources
+ Create roles for your AWS account and define the users or services that can assume them
+ Use existing identities for your enterprise to grant permissions to perform tasks using AWS resources

By using IAM with Kinesis Data Streams, you can control whether users in your organization can perform a task using specific Kinesis Data Streams API actions and whether they can use specific AWS resources\.

If you are developing an application using the Kinesis Client Library \(KCL\), your policy must include permissions for Amazon DynamoDB and Amazon CloudWatch; the KCL uses DynamoDB to track state information for the application, and CloudWatch to send KCL metrics to CloudWatch on your behalf\. For more information about the KCL, see [Developing KCL 1\.x Consumers](developing-consumers-with-kcl.md)\.

For more information about IAM, see the following:
+ [AWS Identity and Access Management \(IAM\)](https://aws.amazon.com/iam/)
+ [Getting Started](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html)
+ [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)

For more information about IAM and Amazon DynamoDB, see [Using IAM to Control Access to Amazon DynamoDB Resources](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/UsingIAMWithDDB.html) in the *Amazon DynamoDB Developer Guide*\. 

For more information about IAM and Amazon CloudWatch, see [Controlling User Access to Your AWS Account](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/UsingIAM.html) in the *Amazon CloudWatch User Guide*\.

**Topics**
+ [Policy Syntax](#policy-syntax)
+ [Actions for Kinesis Data Streams](#kinesis-using-iam-actions)
+ [Amazon Resource Names \(ARNs\) for Kinesis Data Streams](#kinesis-using-iam-arn-format)
+ [Example Policies for Kinesis Data Streams](#kinesis-using-iam-examples)

## Policy Syntax<a name="policy-syntax"></a>

An IAM policy is a JSON document that consists of one or more statements\. Each statement is structured as follows:

```
{
  "Statement":[{
    "Effect":"effect",
    "Action":"action",
    "Resource":"arn",
    "Condition":{
      "condition":{
        "key":"value"
        }
      }
    }
  ]
}
```

There are various elements that make up a statement:
+ **Effect:** The *effect* can be `Allow` or `Deny`\. By default, IAM users don't have permission to use resources and API actions, so all requests are denied\. An explicit allow overrides the default\. An explicit deny overrides any allows\.
+ **Action**: The *action* is the specific API action for which you are granting or denying permission\.
+ **Resource**: The resource that's affected by the action\. To specify a resource in the statement, you need to use its Amazon Resource Name \(ARN\)\.
+ **Condition**: Conditions are optional\. They can be used to control when your policy will be in effect\.

As you create and manage IAM policies, you might want to use the [IAM Policy Generator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-generator) and the [IAM Policy Simulator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)\.

## Actions for Kinesis Data Streams<a name="kinesis-using-iam-actions"></a>

In an IAM policy statement, you can specify any API action from any service that supports IAM\. For Kinesis Data Streams, use the following prefix with the name of the API action: `kinesis:`\. For example: `kinesis:CreateStream`, `kinesis:ListStreams`, and `kinesis:DescribeStream`\.

To specify multiple actions in a single statement, separate them with commas as follows:

```
"Action": ["kinesis:action1", "kinesis:action2"]
```

You can also specify multiple actions using wildcards\. For example, you can specify all actions whose name begins with the word "Get" as follows:

```
"Action": "kinesis:Get*"
```

To specify all Kinesis Data Streams operations, use the \* wildcard as follows:

```
"Action": "kinesis:*"
```

For the complete list of Kinesis Data Streams API actions, see the [Amazon Kinesis API Reference](https://docs.aws.amazon.com/kinesis/latest/APIReference/)\.

## Amazon Resource Names \(ARNs\) for Kinesis Data Streams<a name="kinesis-using-iam-arn-format"></a>

Each IAM policy statement applies to the resources that you specify using their ARNs\.

Use the following ARN resource format for Kinesis data streams:

```
arn:aws:kinesis:region:account-id:stream/stream-name
```

For example:

```
"Resource": arn:aws:kinesis:*:111122223333:stream/my-stream
```

## Example Policies for Kinesis Data Streams<a name="kinesis-using-iam-examples"></a>

The following example policies demonstrate how you could control user access to your Kinesis data streams\.

**Example 1: Allow users to get data from a stream**  
 This policy allows a user or group to perform the `DescribeStream`, `GetShardIterator`, and `GetRecords` operations on the specified stream and `ListStreams` on any stream\. This policy could be applied to users who should be able to get data from a specific stream\.   

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kinesis:Get*",
                "kinesis:DescribeStream"
            ],
            "Resource": [
                "arn:aws:kinesis:us-east-1:111122223333:stream/stream1"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "kinesis:ListStreams"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

**Example 2: Allow users to add data to any stream in the account**  
This policy allows a user or group to use the `PutRecord` operation with any of the account's streams\. This policy could be applied to users that should be able to add data records to all streams in an account\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kinesis:PutRecord"
            ],
            "Resource": [
                "arn:aws:kinesis:us-east-1:111122223333:stream/*"
            ]
        }
    ]
}
```

**Example 3: Allow any Kinesis Data Streams action on a specific stream**  
This policy allows a user or group to use any Kinesis Data Streams operation on the specified stream\. This policy could be applied to users that should have administrative control over a specific stream\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "kinesis:*",
            "Resource": [
                "arn:aws:kinesis:us-east-1:111122223333:stream/stream1"
            ]
        }
    ]
}
```

**Example 4: Allow any Kinesis Data Streams action on any stream**  
This policy allows a user or group to use any Kinesis Data Streams operation on any stream in an account\. Because this policy grants full access to all your streams, you should restrict it to administrators only\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "kinesis:*",
            "Resource": [
                "arn:aws:kinesis:*:111122223333:stream/*"
            ]
        }
    ]
}
```