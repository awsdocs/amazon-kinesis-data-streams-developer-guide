# Verifying and Troubleshooting KMS Key Permissions<a name="sse-troubleshooting"></a>

After enabling encryption on a Kinesis stream, we recommend that you monitor the success of your `putRecord`, `putRecords`, and `getRecords` calls using the following Amazon CloudWatch metrics:

+ `PutRecord.Success`

+ `PutRecords.Success`

+ `GetRecords.Success`