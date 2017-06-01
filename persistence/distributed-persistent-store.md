* [Overview](#section-overview)
* [Usage](#section-usage)
* [Write-Ahead Log File](#section-write-ahead-log-file)
* [Checkpointing](#section-checkpointing)
* [Transactional Guarantees](#section-transactional-guarantees)
* [SQL Support](#section-sql-support)
* [Persistent Store Internals](#section-persistent-store-internals)
* [Example](#section-example)
[block:api-header]
{
  "title": "Overview"
}
[/block]
Apache Ignite Persistent Store is a distributed and transactional disk store that seamlessly integrates with overall page memory architecture as an additional persistence layer giving a way to keep data on Flash, SSD, Intel 3D XPoint and other types of non-volatile storages.

The Persistent Store satisfies all the Apache Ignite guarantees and properties that are supported for a pure in-memory use case. For instance, this is an ACID and ANSI-99 SQL compliant store.  
 
Apache Ignite [Page Memory](doc:page-memory) is tightly coupled with the Persistent Store and starts keeping data and indexes on disk once the store is enabled in a cluster's configuration. As with a pure in-memory use case, every individual cluster node persists only a subset of data and indexes for which the node is either a primary or backup one.

Apache Ignite Persistent Store has the following advantages over 3rd party stores (RDBMS, NoSQL, Hadoop) that can be used as an alternative persistence layer for an Apache Ignite cluster:
* An ability to execute SQL queries over the data that is both in memory and on disk meaning that Apache Ignite can be used as a memory-optimized distributed SQL database.
* No need to have all the data and indexes in memory. The Persistent Store allows storing a superset of data on disk and have only frequently used subsets in memory.
* Instantaneous cluster restarts. If the whole cluster goes down there is no need to warm up the memory preloading data from the Persistent Store. The cluster becomes fully operational once all the cluster nodes are interconnected with each other.
* Data and indexes are stored in a similar format both in memory and on disk that helps to avoid expensive transformations while the data sets are being moved between memory and disk. 
* An ability to create full and incremental cluster snapshots by plugging-in 3rd party solutions.
[block:api-header]
{
  "title": "Usage"
}
[/block]
To enable the distributed Persistent Store, pass an instance of `PersistentStoreConfiguration` to a cluster node configuration: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  <!-- Enabling Apache Ignite Persistent Store. -->\n  <property name=\"persistentStoreConfiguration\">\n    <bean class=\"org.apache.ignite.configuration.PersistentStoreConfiguration\"/>\n  </property>\n\n  <!-- Additional setting. -->\n \n</bean>",
      "language": "xml"
    },
    {
      "code": "// Apache Ignite node configuration.\nIgniteConfiguration cfg = new IgniteConfiguration();\n\n// Persistent Store configuration.\nPersistentStoreConfiguration psCfg = new PersistentStoreConfiguration();\n\n// Enabling the Persistent Store.\ncfg.setPersistentStoreConfiguration(psCfg);\n        \n//Additional parameters.",
      "language": "java"
    }
  ]
}
[/block]
Once this is done, the Persistent Store will be enabled and all the data, as well as indexes, will be stored both in memory and on disk across all the cluster nodes. 
[block:callout]
{
  "type": "success",
  "title": "Persistent Store Root Path",
  "body": "By default, all the data is persisted in the Apache Ignite working directory (`${IGNITE_HOME}/work`). Use `PersistentStoreConfiguration.setPersistentStorePath(...)` method to change the default directory."
}
[/block]
Having the Persistent Store enabled, you're no longer need to fit all the data in RAM. The disk will store all the data and indexes you have while a subset of them will be kept in RAM. This is beneficial when you have limited physical memory resources or wish to store and query historical data in Apache Ignite.

Taking this into account, if a page is not found in RAM, then the page memory will request it from the Persistent Store. The subset of data that is to be stored in the off-heap memory is defined by [eviction policies](https://apacheignite.readme.io/docs/evictions#section-page-based-eviction) you use for memory regions. Plus, pages of backup partitions will be evicted from RAM first giving more space to pages of partitions a node is primary for.
[block:api-header]
{
  "title": "Write-Ahead Log File"
}
[/block]
The Persistent Store creates and maintains a dedicated file for every partition a node is either primary or backup one. However, when a page is updated in physical memory the update is not directly written to a respective partition file because it can affect performance dramatically. It's rather appended to the tail of an Apache Ignite node's write-ahead log (WAL) file.

The purpose of the WAL file is to propagate updates to disk in the fastest way possible and provide a recovery mechanism for scenarios when a single node or the whole cluster goes down. It worth mentioning that every update is uniquely defined cluster-wide, which means that a cluster can always be recovered to the latest successfully committed transaction in case of a crash or restart relying on the content of the WAL file.
[block:callout]
{
  "type": "success",
  "title": "More Details on WAL",
  "body": "Refer to WAL section on [Persistent Store Internal Design page](https://cwiki.apache.org/confluence/display/IGNITE/Persistent+Store+Architecture#PersistentStoreArchitecture-Write-Ahead-Log) to learn more about WAL implementation details in Apache Ignite."
}
[/block]
Use configuration parameters below to alter WAL file related settings:
[block:parameters]
{
  "data": {
    "h-0": "Parameter Name",
    "h-1": "Description",
    "h-2": "Default Value",
    "0-0": "`setWalStorePath(...)`",
    "0-1": "Sets a path to the directory where WAL is stored . If this path is relative, it will be resolved relatively to Ignite work directory.",
    "0-2": "`${IGNITE_HOME}/work`",
    "1-0": "`setWalSegments(...)`",
    "1-1": "Sets a number of WAL segments to work with. For performance reasons, the whole WAL is split into files of fixed length called segments.",
    "2-0": "`setWalSegmentSize(...)`",
    "2-1": "Sets size of a WAL segment.",
    "2-2": "`64 MB`",
    "1-2": "`10`",
    "3-1": "Sets a total number of checkpoints to keep in the WAL history. Refer to the checkpointing section below to learn more about that technique.",
    "3-0": "`setWalHistorySize(...)`",
    "3-2": "`20`",
    "4-0": "`setWalArchivePath(...)`",
    "4-1": "Sets a path for the WAL archive directory. Every WAL segment will be fully copied to this directory before it can be reused for WAL purposes.",
    "4-2": "`${IGNITE_HOME}/work`"
  },
  "cols": 3,
  "rows": 5
}
[/block]

[block:api-header]
{
  "title": "Checkpointing"
}
[/block]
The WAL file is an essential part of the Persistent Store which role is:
* To persist updates on disk int the fastest way possible which is by appending an update record to the end of the file.
* To recover the cluster to a consistent state in the case of a restart or crash.

However, due to the nature of the WAL file, it would constantly grow and it would take significant time to recover the cluster by going over the WAL from the head to the tail if the page memory and Persistent Store did not support a checkpointing process.

The checkpointing is a process of copying dirty pages from RAM to the partition files on disk. A dirty page is a page that was updated in RAM but was not written to a respective partition file on disk (an update was just appended to the WAL file).

This process helps to utilize disk space frugally by having pages in the most up-to-date state on disk and truncating the size of the WAL file by removing those update records from it that are already stored in the partition files.  

The checkpointing is triggered periodically depending on a frequency set in your Persistent Store configuration. See from the table below how this and other checkpointing related parameters can be adjusted for your needs: 
     
[block:parameters]
{
  "data": {
    "h-0": "Parameter Name",
    "h-1": "Description",
    "h-2": "Default Value",
    "0-0": "`setCheckpointingFrequency(...)`",
    "0-1": "Sets the checkpointing frequency which is a minimal interval when the dirty pages will be written to the Persistent Store. If the rate is high, checkpointing will be triggered more frequently.",
    "0-2": "`3` minutes",
    "1-0": "`setCheckpointingPageBufferSize(...)`",
    "1-1": "Sets amount of memory allocated for the checkpointing temporary buffer. The buffer is used to create temporary copies of pages that are being written to disk and being update in parallel while the checkpointing is in progress.",
    "1-2": "`256 MB`",
    "2-0": "`setCheckpointingThreads(...)`",
    "2-1": "Sets a number of threads to use for the checkpointing purposes.",
    "2-2": "`1`"
  },
  "cols": 3,
  "rows": 3
}
[/block]

[block:callout]
{
  "type": "success",
  "title": "More Details on Checkpointing",
  "body": "Refer to checkpointing section on [Persistent Store Architecture page](https://cwiki.apache.org/confluence/display/IGNITE/Persistent+Store+Architecture#PersistentStoreArchitecture-Checkpointing) to learn more about checkpointing implementation details in Apache Ignite."
}
[/block]

[block:api-header]
{
  "title": "Transactional Guarantees"
}
[/block]
The Persistent Store is an ACID-compliant distributed store.

Every transactional update that comes to the store is appended to the WAL first. The update is uniquely defined with an ID. All this means that a cluster can always be recovered to the latest successfully committed transaction or an atomic update ​in case of a crash or restart.
[block:api-header]
{
  "title": "SQL Support"
}
[/block]
The Persistent Store allows using Apache Ignite as a distributed SQL database. The store is a fully ANSI-99 SQL compliant.

There is no need to have all the data in memory if you need to run SQL queries across the cluster. Apache Ignite is able to execute them over the data that is both in memory and on disk. 

Moreover, it's optional to preload data from the Persistent Store to the memory after a cluster's restart. Your applicant can run SQL queries as soon as the cluster is up and running.  
[block:api-header]
{
  "title": "Persistent Store Internals"
}
[/block]
This documentation provides a high-level overview of the Persistent Store needed to start using it in production. If you're curious to get more technical details refer to these documents:
* [Persistent Store Design](https://cwiki.apache.org/confluence/display/IGNITE/Persistent+Store+Overview)
* [Persistent Store Architecture](https://cwiki.apache.org/confluence/display/IGNITE/Persistent+Store+Architecture)
[block:api-header]
{
  "title": "Example"
}
[/block]
To see how the Persistent Store can be used in practice give a try to [this](https://github.com/apache/ignite/tree/ignite-5267/examples/src/main/java/org/apache/ignite/examples/persistentstore) example that is available on GitHub and delivered with every Apache Ignite distribution.