# Step 2: Set Up the AWS Command Line Interface \(AWS CLI\)<a name="setup-awscli"></a>

In this step, you download and configure the AWS CLI to use with Amazon Kinesis Data Analytics for Java Applications\.

**Note**  
The getting started exercises in this guide assume that you are using administrator credentials \(`adminuser`\) in your account to perform the operations\.

**Note**  
If you already have the AWS CLI installed, you might need to upgrade to get the latest functionality\. For more information, see [ Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\. To check the version of the AWS CLI, run the following command:  

```
aws --version
```
The exercises in this tutorial require the following AWS CLI version or later:  

```
aws-cli/1.16.63
```

**To set up the AWS CLI**

1. Download and configure the AWS CLI\. For instructions, see the following topics in the *AWS Command Line Interface User Guide*: 
   + [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html)
   + [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)

1. Add a named profile for the administrator user in the AWS CLI config file\. You use this profile when executing the AWS CLI commands\. For more information about named profiles, see [Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html) in the *AWS Command Line Interface User Guide*\.

   ```
   [profile adminuser]
   aws_access_key_id = adminuser access key ID
   aws_secret_access_key = adminuser secret access key
   region = aws-region
   ```

   For a list of available AWS Regions, see [AWS Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html) in the *Amazon Web Services General Reference*\.

1. Verify the setup by entering the following help command at the command prompt: 

   ```
   aws help
   ```

After you set up an AWS account and the AWS CLI, you can try the next exercise, in which you configure a sample application and test the end\-to\-end setup\.

## Next Step<a name="setting-up-next-step-3"></a>

[ Step 3: Create and Run a Kinesis Data Analytics for Java Application](get-started-exercise.md)