# Supported Platforms<a name="kinesis-kpl-supported-plats"></a>

The KPL is written in C\+\+ and runs as a child process to the main user process\. Precompiled 64\-bit native binaries are bundled with the Java release and are managed by the Java wrapper\.

The Java package runs without the need to install any additional libraries on the following operating systems:

+ Linux distributions with kernel 2\.6\.18 \(September 2006\) and later

+ Apple OS X 10\.9 and later

+ Windows Server 2008 and later

Note the KPL is 64\-bit only\.

## Source Code<a name="w3ab1c11b7b7c17c11"></a>

If the binaries provided in the KPL installation are not sufficient for your environment, the core of the KPL is written as a C\+\+ module\. The source code for the C\+\+ module and the Java interface are released under the Amazon Public License and are available on GitHub at [Kinesis Producer Library](https://github.com/awslabs/amazon-kinesis-producer)\. Although the KPL can be used on any platform for which a recent standards\-compliant C\+\+ compiler and JRE are available, Amazon does not officially support any platform not on the supported platforms list\.