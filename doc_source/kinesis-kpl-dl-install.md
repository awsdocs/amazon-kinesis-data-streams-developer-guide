# Installing the KPL<a name="kinesis-kpl-dl-install"></a>

Amazon provides pre\-built binaries of the C\+\+ Kinesis Producer Library \(KPL\) for macOS, Windows, and recent Linux distributions \(for supported platform details, see the next section\)\. These binaries are packaged as part of Java \.jar files and are automatically invoked and used if you are using Maven to install the package\. To locate the latest versions of the KPL and KCL, use the following Maven search links:
+ [KPL](https://search.maven.org/#search|ga|1|amazon-kinesis-producer)
+ [KCL](https://search.maven.org/#search|ga|1|amazon-kinesis-client)

The Linux binaries have been compiled with the GNU Compiler Collection \(GCC\) and statically linked against libstdc\+\+ on Linux\. They are expected to work on any 64\-bit Linux distribution that includes a glibc version 2\.5 or higher\.

Users of older Linux distributions can build the KPL using the build instructions provided along with the source on GitHub\. To download the KPL from GitHub, see [Kinesis Producer Library](https://github.com/awslabs/amazon-kinesis-producer)\.