# KPL Proxy Configuration<a name="kpl-proxy-configuration"></a>

For applications that cannot directly connect to the internet, all AWS SDK clients support the use of HTTP or HTTPS proxies\. In a typical enterprise environment, all outbound network traffic has to go through proxy servers\. If your application uses Kinesis Producer Library \(KPL\) to collect and send data to AWS in an environment that uses proxy servers, your application will require KPL proxy configuration\. KPL is a high level library built on top of the AWS Kinesis SDK\. It is split into a native process and a wrapper\. The native process performs all of the jobs of processing and sending records, while the wrapper manages the native process and communicates with it\. For more information, see [Implementing Efficient and Reliable Producers with the Amazon Kinesis Producer Library](https://aws.amazon.com/blogs/big-data/implementing-efficient-and-reliable-producers-with-the-amazon-kinesis-producer-library/)\. 

The wrapper is written in Java and the native process is written in C\+\+ with the use of Kinesis SDK\. KPL version 0\.14\.7 and higher now supports proxy configuration in the Java wrapper which can pass all proxy configurations to the native process\. For more information, see [https://github\.com/awslabs/amazon\-kinesis\-producer/releases/tag/v0\.14\.7](The wrapper is written in Java and the native process is written in C++ with the use of Kinesis SDK. KPL version 0.14.7 now supports proxy configuration in the Java wrapper which can pass all proxy configurations to the native process. For more information, see https://github.com/awslabs/amazon-kinesis-producer/releases/tag/v0.14.7)\.

You can use the following code to add proxy configurations to your KPL applications\.

```
KinesisProducerConfiguration configuration = new KinesisProducerConfiguration();
// Next 4 lines used to configure proxy 
configuration.setProxyHost("10.0.0.0"); // required
configuration.setProxyPort(3128); // default port is set to 443
configuration.setProxyUserName("username"); // no default 
configuration.setProxyPassword("password"); // no default

KinesisProducer kinesisProducer = new KinesisProducer(configuration);
```