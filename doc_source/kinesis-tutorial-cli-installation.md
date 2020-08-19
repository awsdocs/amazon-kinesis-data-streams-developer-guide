# Install and Configure the AWS CLI<a name="kinesis-tutorial-cli-installation"></a>

## Install AWS CLI<a name="install-cli"></a>

For detailed steps on how to install the AWS CLI for Windows and for Linux, OS X, and Unix operating systems, see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)\. 

 Use the following command to list available options and services: 

```
aws help
```

You will be using the Kinesis Data Streams service, so you can review the AWS CLI subcommands related to Kinesis Data Streams using the following command:

```
aws kinesis help
```

This command results in output that includes the available Kinesis Data Streams commands:

```
AVAILABLE COMMANDS

       o add-tags-to-stream

       o create-stream

       o delete-stream

       o describe-stream

       o get-records

       o get-shard-iterator

       o help

       o list-streams

       o list-tags-for-stream

       o merge-shards

       o put-record

       o put-records

       o remove-tags-from-stream

       o split-shard

       o wait
```

 This command list corresponds to the Kinesis Data Streams API documented in the [Amazon Kinesis Service API Reference](https://docs.aws.amazon.com/kinesis/latest/APIReference/)\. For example, the `create-stream` command corresponds to the `CreateStream` API action\. 

 The AWS CLI is now successfully installed, but not configured\. This is shown in the next section\. 

## Configure AWS CLI<a name="config-cli"></a>

 For general use, the `aws configure` command is the fastest way to set up your AWS CLI installation\. For more information, see [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.