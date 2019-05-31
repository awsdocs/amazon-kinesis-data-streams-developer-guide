# Step 1: Set Up an AWS Account and Create an Administrator User<a name="setting-up"></a>

Before you use Amazon Kinesis Data Analytics for Java Applications for the first time, complete the following tasks:

1. [Sign Up for AWS](#setting-up-signup)

1. [Create an IAM User](#setting-up-iam)

## Sign Up for AWS<a name="setting-up-signup"></a>

When you sign up for Amazon Web Services \(AWS\), your AWS account is automatically signed up for all services in AWS, including Amazon Kinesis Data Analytics\. You are charged only for the services that you use\.

With Kinesis Data Analytics, you pay only for the resources that you use\. If you are a new AWS customer, you can get started with Kinesis Data Analytics for free\. For more information, see [AWS Free Tier](https://aws.amazon.com/free/)\.

If you already have an AWS account, skip to the next task\. If you don't have an AWS account, follow these steps to create one\.

**To create an AWS account**

1. Open [https://portal\.aws\.amazon\.com/billing/signup](https://portal.aws.amazon.com/billing/signup)\.

1. Follow the online instructions\.

   Part of the sign\-up procedure involves receiving a phone call and entering a verification code on the phone keypad\.

Note your AWS account ID because you'll need it for the next task\.

## Create an IAM User<a name="setting-up-iam"></a>

Services in AWS, such as Amazon Kinesis Data Analytics, require that you provide credentials when you access them\. This is so that the service can determine whether you have permissions to access the resources that are owned by that service\. The AWS Management Console requires that you enter your password\. 

You can create access keys for your AWS account to access the AWS Command Line Interface \(AWS CLI\) or API\. However, we don't recommend that you access AWS using the credentials for your AWS account\. Instead, we recommend that you use AWS Identity and Access Management \(IAM\)\. Create an IAM user, add the user to an IAM group with administrative permissions, and then grant administrative permissions to the IAM user that you created\. You can then access AWS using a special URL and that IAM user's credentials\.

If you signed up for AWS, but you haven't created an IAM user for yourself, you can create one using the IAM console\.

The getting started exercises in this guide assume that you have a user \(`adminuser`\) with administrator permissions\. Follow the procedure to create `adminuser` in your account\.

**To create a group for administrators**

1. Sign in to the AWS Management Console and open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Groups**, and then choose **Create New Group**\.

1. For **Group Name**, enter a name for your group, such as **Administrators**, and then choose **Next Step**\.

1. In the list of policies, select the check box next to the **AdministratorAccess** policy\. You can use the **Filter** menu and the **Search** box to filter the list of policies\.

1. Choose **Next Step**, and then choose **Create Group**\.

Your new group is listed under **Group Name**\.

**To create an IAM user for yourself, add it to the Administrators group, and create a password**

1. In the navigation pane, choose **Users**, and then choose **Add user**\.

1. In the **User name** box, enter a user name\.

1. Choose both **Programmatic access** and **AWS Management Console access**\.

1. Choose **Next: Permissions**\.

1. Select the check box next to the **Administrators** group\. Then choose **Next: Review**\.

1. Choose **Create user**\.

**To sign in as the new IAM user**

1. Sign out of the AWS Management Console\.

1. Use the following URL format to sign in to the console:

   `https://aws_account_number.signin.aws.amazon.com/console/`

   The *aws\_account\_number* is your AWS account ID without any hyphens\. For example, if your AWS account ID is 1234\-5678\-9012, replace *aws\_account\_number* with **123456789012**\. For information about how to find your account number, see [Your AWS Account ID and Its Alias](https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html) in the *IAM User Guide*\.

1. Enter the IAM user name and password that you just created\. When you're signed in, the navigation bar displays *your\_user\_name* @ *your\_aws\_account\_id*\.

**Note**  
If you don't want the URL for your sign\-in page to contain your AWS account ID, you can create an account alias\.

**To create or remove an account alias**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. On the navigation pane, choose **Dashboard**\.

1. Find the IAM users sign\-in link\.

1. To create the alias, choose **Customize**\. Enter the name you want to use for your alias, and then choose **Yes, Create**\.

1. To remove the alias, choose **Customize**, and then choose **Yes, Delete**\. The sign\-in URL reverts to using your AWS account ID\.

To sign in after you create an account alias, use the following URL:

`https://your_account_alias.signin.aws.amazon.com/console/`

To verify the sign\-in link for IAM users for your account, open the IAM console and check under **IAM users sign\-in link** on the dashboard\.

For more information about IAM, see the following:
+ [AWS Identity and Access Management \(IAM\)](https://aws.amazon.com/iam/)
+ [Getting Started](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html)
+ [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)

## Next Step<a name="setting-up-next-step-2"></a>

[Step 2: Set Up the AWS Command Line Interface \(AWS CLI\)](setup-awscli.md)