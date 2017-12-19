# Install and Configure the AWS CLI<a name="kinesis-tutorial-cli-installation"></a>

## Install AWS CLI<a name="install-cli"></a>

Use the following process to install the AWS CLI for Windows and for Linux, OS X, and Unix operating systems\.

### Windows<a name="install-cli-windows"></a>

1. Download the appropriate MSI installer from the Windows section of the [full installation instructions](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the [AWS Command Line Interface User Guide](http://docs.aws.amazon.com/cli/latest/userguide/)\.

1. Run the downloaded MSI installer\.

1. Follow the instructions that appear\.

### Linux, macOS, or Unix<a name="install-cli-unix"></a>

These steps require Python 2\.6\.3 or higher\. If you have any problems, see the [full installation instructions](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the [AWS Command Line Interface User Guide](http://docs.aws.amazon.com/cli/latest/userguide/)\. 

1. Download and run the installation script from the [pip website](https://pip.pypa.io/en/latest/installing.html): 

   ```
   curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
   sudo python get-pip.py
   ```

1. Install the AWS CLI Using Pip\.

   ```
   sudo pip install awscli
   ```

 Use the following command to list available options and services: 

```
aws help
```

You will be using the Kinesis Streams service, so you can review the AWS CLI subcommands related to Kinesis Streams using the following command:

```
aws kinesis help
```

This command results in output that includes the available Kinesis Streams commands:

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

 This command list corresponds to the Kinesis Streams API documented in the [Amazon Kinesis Service API Reference](http://docs.aws.amazon.com/kinesis/latest/APIReference/)\. For example, the `create-stream` command corresponds to the `CreateStream` API action\. 

 The AWS CLI is now successfully installed, but not configured\. This is shown in the next section\. 

## Configure AWS CLI<a name="config-cli"></a>

 For general use, the `aws configure` command is the fastest way to set up your AWS CLI installation\. This is a one\-time setup if your preferences don't change because the AWS CLI remembers your settings between sessions\. 

```
aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

 The AWS CLI will prompt you for four pieces of information\. The AWS access key ID and the AWS secret access key are your account credentials\. If you don't have keys, see [Sign Up for Amazon Web Services](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html#cli-signup)\. 

 The default region is the name of the region you want to make calls against by default\. This is usually the region closest to you, but it can be any region\. 

**Note**  
You must specify an AWS region when using the AWS CLI\. For a list of services and available regions, see [Regions and Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html)\. 

 The default output format can be either JSON, text, or table\. If you don't specify an output format, JSON will be used\. 

 For more information about the files that `aws configure` creates, additional settings, and named profiles, see [Configuring the AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) in the [AWS Command Line Interface User Guide](http://docs.aws.amazon.com/cli/latest/userguide/)\. 