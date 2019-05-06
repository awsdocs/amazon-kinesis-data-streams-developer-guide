# Tagging Your Streams in Amazon Kinesis Data Streams<a name="tagging"></a>

You can assign your own metadata to streams you create in Amazon Kinesis Data Streams in the form of *tags*\. A tag is a key\-value pair that you define for a stream\. Using tags is a simple yet powerful way to manage AWS resources and organize data, including billing data\. 

**Topics**
+ [Tag Basics](#tagging-basics)
+ [Tracking Costs Using Tagging](#tagging-billing)
+ [Tag Restrictions](#tagging-restrictions)
+ [Tagging Streams Using the Kinesis Data Streams Console](#tagging-console)
+ [Tagging Streams Using the AWS CLI](#tagging-cli)
+ [Tagging Streams Using the Kinesis Data Streams API](#tagging-api)

## Tag Basics<a name="tagging-basics"></a>

You use the Kinesis Data Streams console, AWS CLI, or Kinesis Data Streams API to complete the following tasks:
+ Add tags to a stream
+ List the tags for your streams
+ Remove tags from a stream

You can use tags to categorize your streams\. For example, you can categorize streams by purpose, owner, or environment\. Because you define the key and value for each tag, you can create a custom set of categories to meet your specific needs\. For example, you might define a set of tags that helps you track streams by owner and associated application\. Here are several examples of tags:
+ Project: Project name
+ Owner: Name
+ Purpose: Load testing 
+ Application: Application name
+ Environment: Production 

## Tracking Costs Using Tagging<a name="tagging-billing"></a>

You can use tags to categorize and track your AWS costs\. When you apply tags to your AWS resources, including streams, your AWS cost allocation report includes usage and costs aggregated by tags\. You can apply tags that represent business categories \(such as cost centers, application names, or owners\) to organize your costs across multiple services\. For more information, see [Use Cost Allocation Tags for Custom Billing Reports](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html) in the *AWS Billing and Cost Management User Guide*\.

## Tag Restrictions<a name="tagging-restrictions"></a>

The following restrictions apply to tags\.

**Basic restrictions**
+ The maximum number of tags per resource \(stream\) is 50\.
+ Tag keys and values are case\-sensitive\.
+ You can't change or edit tags for a deleted stream\.

**Tag key restrictions**
+ Each tag key must be unique\. If you add a tag with a key that's already in use, your new tag overwrites the existing key\-value pair\. 
+ You can't start a tag key with `aws:` because this prefix is reserved for use by AWS\. AWS creates tags that begin with this prefix on your behalf, but you can't edit or delete them\.
+ Tag keys must be between 1 and 128 Unicode characters in length\.
+ Tag keys must consist of the following characters: Unicode letters, digits, white space, and the following special characters: `_ . / = + - @`\.

**Tag value restrictions**
+ Tag values must be between 0 and 255 Unicode characters in length\.
+ Tag values can be blank\. Otherwise, they must consist of the following characters: Unicode letters, digits, white space, and any of the following special characters: `_ . / = + - @`\.

## Tagging Streams Using the Kinesis Data Streams Console<a name="tagging-console"></a>

You can add, list, and remove tags using the Kinesis Data Streams console\.

**To view the tags for a stream**

1. Open the Kinesis Data Streams console\. In the navigation bar, expand the region selector and select a region\.

1. On the **Stream List** page, select a stream\.

1. On the **Stream Details** page, click the **Tags** tab\.

**To add a tag to a stream**

1. Open the Kinesis Data Streams console\. In the navigation bar, expand the region selector and select a region\.

1. On the **Stream List** page, select a stream\.

1. On the **Stream Details** page, click the **Tags** tab\.

1. Specify the tag key in the **Key** field, optionally specify a tag value in the **Value** field, and then click **Add Tag**\.

   If the **Add Tag** button is not enabled, either the tag key or tag value that you specified don't meet the tag restrictions\. For more information, see [Tag Restrictions](#tagging-restrictions)\.

1. To view your new tag in the list on the **Tags** tab, click the refresh icon\.

**To remove a tag from a stream**

1. Open the Kinesis Data Streams console\. In the navigation bar, expand the region selector and select a region\.

1. On the Stream List page, select a stream\.

1. On the Stream Details page, click the **Tags** tab, and then click the **Remove** icon for the tag\.

1. In the **Delete Tag** dialog box, click **Yes, Delete**\.

## Tagging Streams Using the AWS CLI<a name="tagging-cli"></a>

You can add, list, and remove tags using the AWS CLI\. For examples, see the following documentation\.

 [add\-tags\-to\-stream](https://docs.aws.amazon.com/cli/latest/reference/kinesis/add-tags-to-stream.html)   
Adds or updates tags for the specified stream\.

 [list\-tags\-for\-stream](https://docs.aws.amazon.com/cli/latest/reference/kinesis/list-tags-for-stream.html)  
Lists the tags for the specified stream\.

 [remove\-tags\-from\-stream](https://docs.aws.amazon.com/cli/latest/reference/kinesis/remove-tags-from-stream.html)  
Removes tags from the specified stream\.

## Tagging Streams Using the Kinesis Data Streams API<a name="tagging-api"></a>

You can add, list, and remove tags using the Kinesis Data Streams API\. For examples, see the following documentation:

 [AddTagsToStream](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_AddTagsToStream.html)   
Adds or updates tags for the specified stream\.

 [ListTagsForStream](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListTagsForStream.html)  
Lists the tags for the specified stream\.

 [RemoveTagsFromStream](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_RemoveTagsFromStream.html)  
Removes tags from the specified stream\.