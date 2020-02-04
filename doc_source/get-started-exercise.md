# Step 3: Create and Run a Kinesis Data Analytics for Java Application<a name="get-started-exercise"></a>

In this exercise, you create a Kinesis Data Analytics for Java application with data streams as a source and a sink\.

**Topics**
+ [Create Two Amazon Kinesis Data Streams](#get-started-exercise-1)
+ [Write Sample Records to the Input Stream](#get-started-exercise-2)
+ [Download and Examine the Apache Flink Streaming Java Code](#get-started-exercise-5)
+ [Compile the Application Code](#get-started-exercise-5.5)
+ [Upload the Apache Flink Streaming Java Code](#get-started-exercise-6)
+ [Create and Run the Kinesis Data Analytics Application](#get-started-exercise-7)

## Create Two Amazon Kinesis Data Streams<a name="get-started-exercise-1"></a>

Before you create a Kinesis Data Analytics for Java application for this exercise, create two Kinesis data streams \(`ExampleInputStream` and `ExampleOutputStream`\)\. Your application uses these streams for the application source and destination streams\.

You can create these streams using either the Amazon Kinesis console or the following AWS CLI command\. For console instructions, see [Creating and Updating Data Streams](https://docs.aws.amazon.com/kinesis/latest/dev/amazon-kinesis-streams.html)\. 

**To create the data streams \(AWS CLI\)**

1. To create the first stream \(`ExampleInputStream`\), use the following Amazon Kinesis `create-stream` AWS CLI command\.

   ```
   $ aws kinesis create-stream \
   --stream-name ExampleInputStream \
   --shard-count 1 \
   --region us-west-2 \
   --profile adminuser
   ```

1. To create the second stream that the application uses to write output, run the same command, changing the stream name to `ExampleOutputStream`\.

   ```
   $ aws kinesis create-stream \
   --stream-name ExampleOutputStream \
   --shard-count 1 \
   --region us-west-2 \
   --profile adminuser
   ```

## Write Sample Records to the Input Stream<a name="get-started-exercise-2"></a>

In this section, you use a Python script to write sample records to the stream for the application to process\.

**Note**  
This section requires the [AWS SDK for Python \(Boto\)](https://aws.amazon.com/developers/getting-started/python/)\.

1. Create a file named `stock.py` with the following contents:

   ```
    
   import json
   import boto3
   import random
   import datetime
   
   kinesis = boto3.client('kinesis')
   def getReferrer():
       data = {}
       now = datetime.datetime.now()
       str_now = now.isoformat()
       data['EVENT_TIME'] = str_now
       data['TICKER'] = random.choice(['AAPL', 'AMZN', 'MSFT', 'INTC', 'TBV'])
       price = random.random() * 100
       data['PRICE'] = round(price, 2)
       return data
   
   while True:
           data = json.dumps(getReferrer())
           print(data)
           kinesis.put_record(
                   StreamName="ExampleInputStream",
                   Data=data,
                   PartitionKey="partitionkey")
   ```

1. Later in the tutorial, you run the `stock.py` script to send data to the application\. 

   ```
   $ python stock.py
   ```

## Download and Examine the Apache Flink Streaming Java Code<a name="get-started-exercise-5"></a>

The Java application code for this examples is available from GitHub\. To download the application code, do the following:

1. Clone the remote repository with the following command:

   ```
   git clone https://github.com/aws-samples/amazon-kinesis-data-analytics-java-examples.git
   ```

1. Navigate to the `GettingStarted` directory\.

The application code is located in the `CustomSinkStreamingJob.java` and `CloudWatchLogSink.java` files\. Note the following about the application code:
+ The application uses a Kinesis source to read from the source stream\. The following snippet creates the Kinesis sink:

  ```
  return env.addSource(new FlinkKinesisConsumer<>(inputStreamName,
                  new SimpleStringSchema(), inputProperties));
  ```

## Compile the Application Code<a name="get-started-exercise-5.5"></a>

In this section, you use the Apache Maven compiler to create the Java code for the application\. For information about installing Apache Maven and the Java Development Kit \(JDK\), see [Prerequisites for Completing the Exercises](tutorial-stock-data.md#setting-up-prerequisites)\.

Your Java application requires the following components:
+ A [Project Object Model \(pom\.xml\)](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html) file\. This file contains information about the application's configuration and dependencies, including the Kinesis Data Analytics for Java Applications libraries\.
+ A `main` method that contains the application's logic\.

**Note**  
**In order to use the Kinesis connector for the following application, you need to download the source code for the connector and build it as described in the [Apache Flink documentation](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/connectors/kinesis.html)\.**

**To create and compile the application code**

1. Create a Java/Maven application in your development environment\. For information about creating an application, see the documentation for your development environment:
   + [ Creating your first Java project \(Eclipse Java Neon\)](https://help.eclipse.org/neon/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm)
   + [ Creating, Running and Packaging Your First Java Application \(IntelliJ Idea\)](https://www.jetbrains.com/help/idea/creating-and-running-your-first-java-application.html)

1. Use the following code for a file named `StreamingJob.java`\. 

   ```
    
   package com.amazonaws.services.kinesisanalytics;
   
   import com.amazonaws.services.kinesisanalytics.runtime.KinesisAnalyticsRuntime;
   import org.apache.flink.api.common.serialization.SimpleStringSchema;
   import org.apache.flink.streaming.api.datastream.DataStream;
   import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
   import org.apache.flink.streaming.connectors.kinesis.FlinkKinesisConsumer;
   import org.apache.flink.streaming.connectors.kinesis.FlinkKinesisProducer;
   import org.apache.flink.streaming.connectors.kinesis.config.ConsumerConfigConstants;
   
   import java.io.IOException;
   import java.util.Map;
   import java.util.Properties;
   
   
   
   public class StreamingJob {
   
       private static final String region = "us-east-1";
       private static final String inputStreamName = "ExampleInputStream";
       private static final String outputStreamName = "ExampleOutputStream";
   
       private static DataStream<String> createSourceFromStaticConfig(StreamExecutionEnvironment env) {
           Properties inputProperties = new Properties();
           inputProperties.setProperty(ConsumerConfigConstants.AWS_REGION, region);
           inputProperties.setProperty(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST");
   
           return env.addSource(new FlinkKinesisConsumer<>(inputStreamName, new SimpleStringSchema(), inputProperties));
       }
       
       private static DataStream<String> createSourceFromApplicationProperties(StreamExecutionEnvironment env) throws IOException {
           Map<String, Properties> applicationProperties = KinesisAnalyticsRuntime.getApplicationProperties();
           return env.addSource(new FlinkKinesisConsumer<>(inputStreamName, new SimpleStringSchema(),
                   applicationProperties.get("ConsumerConfigProperties")));
       }
   
       private static FlinkKinesisProducer<String> createSinkFromStaticConfig() {
           Properties outputProperties = new Properties();
           outputProperties.setProperty(ConsumerConfigConstants.AWS_REGION, region);
           outputProperties.setProperty("AggregationEnabled", "false");
   
           FlinkKinesisProducer<String> sink = new FlinkKinesisProducer<>(new SimpleStringSchema(), outputProperties);
           sink.setDefaultStream(outputStreamName);
           sink.setDefaultPartition("0");
           return sink;
       }
       
       private static FlinkKinesisProducer<String> createSinkFromApplicationProperties() throws IOException {
           Map<String, Properties> applicationProperties = KinesisAnalyticsRuntime.getApplicationProperties();
           FlinkKinesisProducer<String> sink = new FlinkKinesisProducer<>(new SimpleStringSchema(),
                   applicationProperties.get("ProducerConfigProperties"));
   
           sink.setDefaultStream(outputStreamName);
           sink.setDefaultPartition("0");
           return sink;
       }
   
       public static void main(String[] args) throws Exception {
           // set up the streaming execution environment
           final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
   
            
            /* if you would like to use runtime configuration properties, uncomment the lines below
            * DataStream<String> input = createSourceFromApplicationProperties(env);
            */
            
           DataStream<String> input = createSourceFromStaticConfig(env);
   
   
            /* if you would like to use runtime configuration properties, uncomment the lines below
            * input.addSink(createSinkFromApplicationProperties())
            */
            
           input.addSink(createSinkFromStaticConfig());
   
           env.execute("Flink Streaming Java API Skeleton");
       }
   }
   ```

   Note the following about the preceding code example:
   + This file contains the `main` method that defines the application's functionality\.
   + Your application creates source and sink connectors to access external resources using a `StreamExecutionEnvironment` object\. 
   + The application creates source and sink connectors using static properties\. To use dynamic application properties, use the `createSourceFromApplicationProperties` and `createSinkFromApplicationProperties` methods to create the connectors\. These methods read the application's properties to configure the connectors\.

1. To use your application code, you compile and package it into a JAR file\. You can compile and package your code in one of two ways:
   + Use the command line Maven tool\. Create your JAR file by running the following command in the directory that contains the `pom.xml` file:

     ```
     mvn package
     ```
   + Use your development environment\. See your development environment documentation for details\.

   You can either upload your package as a JAR file, or you can compress your package and upload it as a ZIP file\. If you create your application using the AWS CLI, you specify your code content type \(JAR or ZIP\)\.

1. If there are errors while compiling, verify that your `JAVA_HOME` environment variable is correctly set\.

If the application compiles successfully, the following file is created:

`target/java-getting-started-1.0.jar`

## Upload the Apache Flink Streaming Java Code<a name="get-started-exercise-6"></a>

In this section, you create an Amazon Simple Storage Service \(Amazon S3\) bucket and upload your application code\.

**To upload the application code**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Choose **Create bucket**\.

1. Enter **ka\-app\-code\-*<username>*** in the **Bucket name** field\. Add a suffix to the bucket name, such as your user name, to make it globally unique\. Choose **Next**\.

1. In the **Configure options** step, keep the settings as they are, and choose **Next**\.

1. In the **Set permissions** step, keep the settings as they are, and choose **Next**\.

1. Choose **Create bucket**\.

1. In the Amazon S3 console, choose the **ka\-app\-code\-*<username>*** bucket, and choose **Upload**\.

1. In the **Select files** step, choose **Add files**\. Navigate to the `java-getting-started-1.0.jar` file that you created in the previous step\. Choose **Next**\.

1. In the **Set permissions** step, keep the settings as they are\. Choose **Next**\.

1. In the **Set properties** step, keep the settings as they are\. Choose **Upload**\.

Your application code is now stored in an Amazon S3 bucket where your application can access it\.

## Create and Run the Kinesis Data Analytics Application<a name="get-started-exercise-7"></a>

You can create and run a Kinesis Data Analytics for Java application using either the console or the AWS CLI\.

**Note**  
When you create the application using the console, your AWS Identity and Access Management \(IAM\) and Amazon CloudWatch Logs resources are created for you\. When you create the application using the AWS CLI, you create these resources separately\.

**Topics**
+ [Create and Run the Application \(Console\)](#get-started-exercise-7-console)
+ [Create and Run the Application \(AWS CLI\)](#get-started-exercise-7-cli)

### Create and Run the Application \(Console\)<a name="get-started-exercise-7-console"></a>

Follow these steps to create, configure, update, and run the application using the console\.

#### Create the Application<a name="get-started-exercise-7-console-create"></a>

1. Open the Kinesis console at [https://console\.aws\.amazon\.com/kinesis](https://console.aws.amazon.com/kinesis)\.

1. On the Amazon Kinesis dashboard, choose **Create analytics application**\.

1. On the **Kinesis Analytics \- Create application** page, provide the application details as follows:
   + For **Application name**, enter **MyApplication**\.
   + For **Description**, enter **My java test app**\.
   + For **Runtime**, choose **Apache Flink 1\.6**\.

1. For **Access permissions**, choose **Create / update IAM role `kinesis-analytics-MyApplication-us-west-2`**\.  
![\[Console screenshot showing the settings on the create application page.\]](http://docs.aws.amazon.com/streams/latest/dev/images/create_01.png)

1. Choose **Create application**\.

**Note**  
When you create a Kinesis Data Analytics for Java application using the console, you have the option of having an IAM role and policy created for your application\. Your application uses this role and policy to access its dependent resources\. These IAM resources are named using your application name and Region as follows:  
Policy: `kinesis-analytics-service-MyApplication-us-west-2`
Role: `kinesis-analytics-MyApplication-us-west-2`

#### Edit the IAM Policy<a name="get-started-exercise-7-console-iam"></a>

Edit the IAM policy to add permissions to access the Kinesis data streams\.

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. Choose **Policies**\. Choose the **`kinesis-analytics-service-MyApplication-us-west-2`** policy that the console created for you in the previous section\. 

1. On the **Summary** page, choose **Edit policy**\. Choose the **JSON** tab\.

1. Add the highlighted section of the following policy example to the policy\. Replace the sample account IDs \(*012345678901*\) with your account ID\.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "ReadCode",
               "Effect": "Allow",
               "Action": [
                   "s3:GetObject",
                   "s3:GetObjectVersion"
               ],
               "Resource": [
                   "arn:aws:s3:::ka-app-code-username/java-getting-started-1.0.jar"
               ]
           },
           {
               "Sid": "ListCloudwatchLogGroups",
               "Effect": "Allow",
               "Action": [
                   "logs:DescribeLogGroups"
               ],
               "Resource": [
                   "arn:aws:logs:us-west-2:012345678901:log-group:*"
               ]
           },
           {
               "Sid": "ListCloudwatchLogStreams",
               "Effect": "Allow",
               "Action": [
                   "logs:DescribeLogStreams"
               ],
               "Resource": [
                   "arn:aws:logs:us-west-2:012345678901:log-group:/aws/kinesis-analytics/MyApplication:log-stream:*"
               ]
           },
           {
               "Sid": "PutCloudwatchLogs",
               "Effect": "Allow",
               "Action": [
                   "logs:PutLogEvents"
               ],
               "Resource": [
                   "arn:aws:logs:us-west-2:012345678901:log-group:/aws/kinesis-analytics/MyApplication:log-stream:kinesis-analytics-log-stream"
               ]
           },
           {
               "Sid": "ReadInputStream",
               "Effect": "Allow",
               "Action": "kinesis:*",
               "Resource": "arn:aws:kinesis:us-west-2:012345678901:stream/ExampleInputStream"
           },
           {
               "Sid": "WriteOutputStream",
               "Effect": "Allow",
               "Action": "kinesis:*",
               "Resource": "arn:aws:kinesis:us-west-2:012345678901:stream/ExampleOutputStream"
           }
       ]
   }
   ```

#### Configure the Application<a name="get-started-exercise-7-console-configure"></a>

1. On the **MyApplication** page, choose **Configure**\.  
![\[Screenshot showing the MyApplication page and the configure and run buttons.\]](http://docs.aws.amazon.com/streams/latest/dev/images/create_02.png)

1. On the **Configure application** page, provide the **Code location**:
   + For **Amazon S3 bucket**, enter **ka\-app\-code\-*<username>***\.
   + For **Path to Amazon S3 object**, enter **java\-getting\-started\-1\.0\.jar**\.

1. Under **Access to application resources**, for **Access permissions**, choose **Create / update IAM role `kinesis-analytics-MyApplication-us-west-2`**\.

1. Under **Properties**, for **Group ID**, enter **ProducerConfigProperties**\.

1. Enter the following application properties and values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/streams/latest/dev/get-started-exercise.html)

1. Under **Monitoring**, ensure that the **Monitoring metrics level** is set to **Application**\.

1. For **CloudWatch logging**, select the **Enable** check box\.

1. Choose **Update**\.  
![\[Screenshot of the Configure application page with the settings as described in this procedure.\]](http://docs.aws.amazon.com/streams/latest/dev/images/create_03.png)

**Note**  
When you choose to enable CloudWatch logging, Kinesis Data Analytics creates a log group and log stream for you\. The names of these resources are as follows:   
Log group: `/aws/kinesis-analytics/MyApplication`
Log stream: `kinesis-analytics-log-stream`

#### Run the Application<a name="get-started-exercise-7-console-run"></a>

1. On the **MyApplication** page, choose **Run**\. Confirm the action\.  
![\[Screenshot of the MyApplication page and the run button.\]](http://docs.aws.amazon.com/streams/latest/dev/images/create_04.png)

1. When the application is running, refresh the page\. The console shows the **Application graph**\.

![\[Screenshot of the Application graph.\]](http://docs.aws.amazon.com/streams/latest/dev/images/create_04_1.png)

#### Stop the Application<a name="get-started-exercise-7-console-stop"></a>

On the **MyApplication** page, choose **Stop**\. Confirm the action\.

![\[Screenshot of the MyApplication page and the stop button.\]](http://docs.aws.amazon.com/streams/latest/dev/images/create_05.png)

#### Update the Application<a name="get-started-exercise-7-console-update"></a>

Using the console, you can update application settings such as application properties, monitoring settings, and the location or file name of the application JAR\. You can also reload the application JAR from the Amazon S3 bucket if you need to update the application code\.

On the **MyApplication** page, choose **Configure**\. Update the application settings and choose **Update**\.

### Create and Run the Application \(AWS CLI\)<a name="get-started-exercise-7-cli"></a>

In this section, you use the AWS CLI to create and run the Kinesis Data Analytics application\. Kinesis Data Analytics for Java Applications uses the `kinesisanalyticsv2` AWS CLI command to create and interact with Kinesis Data Analytics applications\.

#### Create a Permissions Policy<a name="get-started-exercise-7-cli-policy"></a>

First, you create a permissions policy with two statements: one that grants permissions for the `read` action on the source stream, and another that grants permissions for `write` actions on the sink stream\. You then attach the policy to an IAM role \(which you create in the next section\)\. Thus, when Kinesis Data Analytics assumes the role, the service has the necessary permissions to read from the source stream and write to the sink stream\.

Use the following code to create the `KAReadSourceStreamWriteSinkStream` permissions policy\. Replace `username` with the user name that you used to create the Amazon S3 bucket to store the application code\. Replace the account ID in the Amazon Resource Names \(ARNs\) \(`012345678901`\) with your account ID\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "S3",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": ["arn:aws:s3:::ka-app-code-username",
                "arn:aws:s3:::ka-app-code-username/*"
            ]
        },
        {
            "Sid": "ReadInputStream",
            "Effect": "Allow",
            "Action": "kinesis:*",
            "Resource": "arn:aws:kinesis:us-west-2:012345678901:stream/ExampleInputStream"
        },
        {
            "Sid": "WriteOutputStream",
            "Effect": "Allow",
            "Action": "kinesis:*",
            "Resource": "arn:aws:kinesis:us-west-2:012345678901:stream/ExampleOutputStream"
        }
    ]
}
```

For step\-by\-step instructions to create a permissions policy, see [Tutorial: Create and Attach Your First Customer Managed Policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_managed-policies.html#part-two-create-policy) in the *IAM User Guide*\.

**Note**  
To access other AWS services, you can use the AWS SDK for Java\. Kinesis Data Analytics automatically sets the credentials required by the SDK to those of the service execution IAM role that is associated with your application\. No additional steps are needed\.

#### Create an IAM Role<a name="get-started-exercise-7-cli-role"></a>

In this section, you create an IAM role that the Kinesis Data Analytics for Java application can assume to read a source stream and write to the sink stream\.

Kinesis Data Analytics cannot access your stream without permissions\. You grant these permissions via an IAM role\. Each IAM role has two policies attached\. The trust policy grants Kinesis Data Analytics permission to assume the role, and the permissions policy determines what Kinesis Data Analytics can do after assuming the role\.

You attach the permissions policy that you created in the preceding section to this role\.

**To create an IAM role**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**, **Create Role**\.

1. Under **Select type of trusted identity**, choose **AWS Service**\. Under **Choose the service that will use this role**, choose **Kinesis**\. Under **Select your use case**, choose **Kinesis Analytics**\.

   Choose **Next: Permissions**\.

1. On the **Attach permissions policies** page, choose **Next: Review**\. You attach permissions policies after you create the role\.

1. On the **Create role** page, enter **KA\-stream\-rw\-role** for the **Role name**\. Choose **Create role**\.

   Now you have created a new IAM role called `KA-stream-rw-role`\. Next, you update the trust and permissions policies for the role\.

1. Attach the permissions policy to the role\.
**Note**  
For this exercise, Kinesis Data Analytics assumes this role for both reading data from a Kinesis data stream \(source\) and writing output to another Kinesis data stream\. So you attach the policy that you created in the previous step, [Create a Permissions Policy](#get-started-exercise-7-cli-policy)\.

   1. On the **Summary** page, choose the **Permissions** tab\.

   1. Choose **Attach Policies**\.

   1. In the search box, enter **KAReadSourceStreamWriteSinkStream** \(the policy that you created in the previous section\)\.

   1. Choose the **KAReadInputStreamWriteOutputStream** policy, and choose **Attach policy**\.

You now have created the service execution role that your application uses to access resources\. Make a note of the ARN of the new role\.

For step\-by\-step instructions for creating a role, see [Creating an IAM Role \(Console\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html#roles-creatingrole-user-console) in the *IAM User Guide*\.

#### Create the Kinesis Data Analytics Application<a name="get-started-exercise-7-cli-create"></a>

1. Save the following JSON code to a file named `create_request.json`\. Replace the sample role ARN with the ARN for the role that you created previously\. Replace the bucket ARN suffix \(`username`\) with the suffix that you chose in the previous section\. Replace the sample account ID \(`012345678901`\) in the service execution role with your account ID\.

   ```
   {
       "ApplicationName": "test",
       "ApplicationDescription": "my java test app",
       "RuntimeEnvironment": "FLINK-1_6",
       "ServiceExecutionRole": "arn:aws:iam::012345678901:role/KA-stream-rw-role",
       "ApplicationConfiguration": {
           "ApplicationCodeConfiguration": {
               "CodeContent": {
                   "S3ContentLocation": {
                       "BucketARN": "arn:aws:s3:::ka-app-code-username",
                       "FileKey": "java-getting-started-1.0.jar"
                   }
               },
               "CodeContentType": "ZIPFILE"
           },
           "EnvironmentProperties":  { 
            "PropertyGroups": [ 
               { 
                  "PropertyGroupId": "ProducerConfigProperties",
                  "PropertyMap" : {
                       "flink.stream.initpos" : "LATEST",
                       "aws.region" : "us-west-2",
                       "AggregationEnabled" : "false"
                  }
               },
               { 
                  "PropertyGroupId": "ConsumerConfigProperties",
                  "PropertyMap" : {
                       "aws.region" : "us-west-2"
                  }
               }
            ]
         }
       }
   }
   ```

1. Execute the [https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_CreateApplication.html](https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_CreateApplication.html) action with the preceding request to create the application: 

   ```
   aws kinesisanalyticsv2 create-application --cli-input-json file://create_request.json
   ```

The application is now created\. You start the application in the next step\.

#### Start the Application<a name="get-started-exercise-7-cli-start"></a>

In this section, you use the [https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StartApplication.html](https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StartApplication.html) action to start the application\.

**To start the application**

1. Save the following JSON code to a file named `start_request.json`\.

   ```
   {
       "ApplicationName": "test",
       "RunConfiguration": {
           "ApplicationRestoreConfiguration": { 
            "ApplicationRestoreType": "RESTORE_FROM_LATEST_SNAPSHOT"
            }
       }
   }
   ```

1. Execute the [https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StartApplication.html](https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StartApplication.html) action with the preceding request to start the application:

   ```
   aws kinesisanalyticsv2 start-application --cli-input-json file://start_request.json
   ```

The application is now running\. You can check the Kinesis Data Analytics metrics on the Amazon CloudWatch console to verify that the application is working\.

#### Stop the Application<a name="get-started-exercise-7-cli-stop"></a>

In this section, you use the [https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StopApplication.html](https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StopApplication.html) action to stop the application\.

**To stop the application**

1. Save the following JSON code to a file named `stop_request.json`\.

   ```
   {"ApplicationName": "test"
   }
   ```

1. Execute the [https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StopApplication.html](https://docs.aws.amazon.com/kinesisanalytics/latest/apiv2/API_StopApplication.html) action with the following request to stop the application:

   ```
   aws kinesisanalyticsv2 stop-application --cli-input-json file://stop_request.json
   ```

The application is now stopped\.