# Integrating the KPL with Producer Code<a name="kinesis-kpl-integration"></a>

The Kinesis Producer Library \(KPL\) runs in a separate process, and communicates with your parent user process using IPC\. This architecture is sometimes called a [microservice](http://en.wikipedia.org/wiki/Microservices), and is chosen for two main reasons:

**1\) Your user process will not crash even if the KPL crashes**  
Your process could have tasks unrelated to Kinesis Data Streams, and may be able to continue operation even if the KPL crashes\. It is also possible for your parent user process to restart the KPL and recover to a fully working state \(this functionality is in the official wrappers\)\.

An example is a web server that sends metrics to Kinesis Data Streams; the server can continue serving pages even if the Kinesis Data Streams part has stopped working\. Crashing the whole server because of a bug in the KPL would therefore cause an unnecessary outage\.

**2\) Arbitrary clients can be supported**  
There are always customers who use languages other than the ones officially supported\. These customers should also be able to use the KPL easily\.

## Recommended Usage Matrix<a name="kinesis-kpl-integration-usage"></a>

The following usage matrix enumerates the recommended settings for different users and advises you about whether and how you should use the KPL\. Keep in mind that if aggregation is enabled, de\-aggregation must also be used to extract your records on the consumer side\. 


| Producer side language | Consumer side language | KCL Version | Checkpoint logic | Can you use the KPL? | Caveats | 
| --- | --- | --- | --- | --- | --- | 
| Anything but Java | \* | \* | \* | No | N/A | 
| Java | Java | Uses Java SDK directly | N/A | Yes | If aggregation is used, you have to use the provided de\-aggregation library after GetRecords calls\. | 
| Java | Anything but Java | Uses SDK directly | N/A | Yes | Must disable aggregation\.  | 
| Java | Java | 1\.3\.x | N/A | Yes | Must disable aggregation\. | 
| Java | Java  | 1\.4\.x | Calls checkpoint without any arguments | Yes | None | 
| Java | Java | 1\.4\.x | Calls checkpoint with an explicit sequence number | Yes | Either disable aggregation, or change the code to use extended sequence numbers for checkpointing\. | 
| Java | Anything but Java  | 1\.3\.x \+ Multilanguage daemon \+ language\-specific wrapper | N/A | Yes | Must disable aggregation\.  | 