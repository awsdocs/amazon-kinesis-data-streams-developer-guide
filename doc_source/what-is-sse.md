# What Is Server\-Side Encryption for Kinesis Data Streams?<a name="what-is-sse"></a>

Server\-side encryption is a feature in Amazon Kinesis Data Streams that automatically encrypts data before it's at rest by using an AWS KMS customer master key \(CMK\) you specify\. Data is encrypted before it's written to the Kinesis stream storage layer, and decrypted after it’s retrieved from storage\. As a result, your data is encrypted at rest within the Kinesis Data Streams service\. This allows you to meet strict regulatory requirements and enhance the security of your data\.

With server\-side encryption, your Kinesis stream producers and consumers don't need to manage master keys or cryptographic operations\. Your data is automatically encrypted as it enters and leaves the Kinesis Data Streams service, so your data at rest is encrypted\. AWS KMS provides all the master keys that are used by the server\-side encryption feature\. AWS KMS makes it easy to use a CMK for Kinesis that is managed by AWS, a user\-specified AWS KMS CMK, or a master key imported into the AWS KMS service\.

**Note**  
Server\-side encryption encrypts incoming data only after encryption is enabled\. Preexisting data in an unencrypted stream is not encrypted after server\-side encryption is enabled\. 