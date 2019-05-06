# Developing a Kinesis Client Library Consumer in Ruby<a name="kinesis-record-processor-implementation-app-ruby"></a>

You can use the Kinesis Client Library \(KCL\) to build applications that process data from your Kinesis data streams\. The Kinesis Client Library is available in multiple languages\. This topic discusses Ruby\.

The KCL is a Java library; support for languages other than Java is provided using a multi\-language interface called the *MultiLangDaemon*\. This daemon is Java\-based and runs in the background when you are using a KCL language other than Java\. Therefore, if you install the KCL for Ruby and write your consumer app entirely in Ruby, you still need Java installed on your system because of the MultiLangDaemon\. Further, MultiLangDaemon has some default settings you may need to customize for your use case, for example, the AWS Region that it connects to\. For more information about the MultiLangDaemon on GitHub, go to the [KCL MultiLangDaemon project](https://github.com/awslabs/amazon-kinesis-client/tree/v1.x/src/main/java/com/amazonaws/services/kinesis/multilang) page\.

To download the Ruby KCL from GitHub, go to [Kinesis Client Library \(Ruby\)](https://github.com/awslabs/amazon-kinesis-client-ruby)\. To download sample code for a Ruby KCL consumer application, go to the [KCL for Ruby sample project](https://github.com/awslabs/amazon-kinesis-client-ruby/tree/master/samples) page on GitHub\.

For more information about the KCL Ruby support library, see [KCL Ruby Gems Documentation](http://www.rubydoc.info/gems/aws-kclrb)\.