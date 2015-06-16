--------------
title: Hadoop Accelerator
excerpt: In-Memory Plug-n-Play Hadoop Accelerator
--------------

Apache Ignite Hadoop Accelerator provides a set of components allowing for in-memory Hadoop job execution and file system operations. 
[block:api-header]
{
  "type": "basic",
  "title": "MapReduce"
}
[/block]
Hadoop Accelerator ships with alternate high-performant implementation of job tracker which replaces standard Hadoop MapReduce. Use it to boost your Hadoop MapReduce job execution performance.

[More Info](doc:map-reduce) 
[block:api-header]
{
  "type": "basic",
  "title": "IGFS - In-Memory FileSystem"
}
[/block]
Hadoop Accelerator ships with an implementation of Hadoop `FileSystem` which stores file system data in-memory using distributed Ignite File System (`IGFS`).  Use it to minimize disk IO and improve performance of any file system operations.

[More Info](doc:file-system)
[block:api-header]
{
  "type": "basic",
  "title": "Secondary File System"
}
[/block]
Hadoop Accelerator ships with an implementation of `SecondaryFileSystem`. This implementation can be injected into existing IGFS allowing for read-through and write-through behavior over any other Hadoop `FileSystem` implementation (e.g. `HDFS`). Use it if you want your `IGFS` to become an in-memory caching layer over disk-based `HDFS` or any other Hadoop-compliant file system.

[More Info](doc:igfs-secondary-file-system)
[block:api-header]
{
  "type": "basic",
  "title": "Supported Hadoop distributions"
}
[/block]
Apache Ignite Hadoop Accelerator can be used with a number of Hadoop distributions. Each distribution may require specific installation steps. 
See the following installation guides for more information:
  * [Installing on Apache Hadoop](doc:installing-on-apache-hadoop)
  * [Installing on Cloudera CDH](doc:installing-on-cloudera-cdh)
  * [Installing on Hortonworks HDP](doc:installing-on-hortonworks-hdp)
  * [Installing on Apache BigTop](doc:installing-on-apache-bigtop)