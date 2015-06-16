Apache Ignite Hadoop Accelerator provides a set of components allowing for in-memory Hadoop job execution and file system operations.

Each component can be used independently or with conjunction with others. That is, you can use Apache Ignite job tracker over HDFS, Hadoop job tracker over Apache Ignite file system, etc..
[block:api-header]
{
  "type": "basic",
  "title": "MapReduce"
}
[/block]
Apache Ignite Hadoop Accelerator ships with alternate high-performant implementation of job tracker which replaces standard Hadoop one. 
Use it to boost your job execution performance.
[block:api-header]
{
  "type": "basic",
  "title": "FileSystem"
}
[/block]
Apache Ignite Hadoop Accelerator ships with an implementation of Hadoop `FileSystem` which stores file system data in-memory using Ignite File System (`IGFS`). 
Use it to minimize disk IO and improve performance of any file system operations.
[block:api-header]
{
  "type": "basic",
  "title": "IGFS Secondary File System"
}
[/block]
Apache Ignite Hadoop Accelerator ships with implementatoin of `IGFS` `SecondaryFileSystem`. This implementation can be injected into existing IGFS allowing for in-memory data to be read-through and write-through any other Hadoop `FileSystem` implementation (e.g. `HDFS`). 
Use it if you want to take advantage of in-memory data processing while still having persistence.
[block:api-header]
{
  "type": "basic",
  "title": "Supported Hadoop distributions"
}
[/block]
Apache Ignite Hadoop Accelerator can be used with a number of Hadoop distributions. Each distribution may require specific installation steps. 
See the following installation guides for more information:
  * `Apache Hadoop`: http://apacheignite.readme.io/v1.0/docs/installing-on-apache-hadoop
  * `Cloudera CDH`: http://apacheignite.readme.io/v1.0/docs/installing-on-cloudera-cdh
  * `Hortonworks HDP`: http://apacheignite.readme.io/v1.0/docs/installing-on-hortonworks-hdp
  * `Apache BigTop`: http://apacheignite.readme.io/v1.0/docs/installing-on-apache-bigtop