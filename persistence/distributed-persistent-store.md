* [Overview](#section-overview)
* [Usage](#section-usage)
* [Write-Ahead Log File](#section-write-ahead-log-file)
* [Checkpointing](#section-checkpointing) 
* [Example](#section-example)
[block:api-header]
{
  "title": "Overview"
}
[/block]
Apache Ignite Persistent Store is a distributed and transactional disk store that seamlessly integrates with overall memory architecture as an additional memory layer giving a way to persist data on Flash, SSD, Intel 3D XPoint and other types of non-volatile storages and devices.

The Persistent Store satisfies all the Apache Ignite guarantees and properties that are supported for a pure in-memory use case. For instance, this is an ACID and ANSI-99 SQL compliant store.  
 
Apache Ignite [Page Memory](doc:page-memory) is tightly coupled with the Persistent Store and starts keeping data and indexes on disk once the store is enabled in a cluster's configuration. As with a pure in-memory use case, every individual cluster node persists only a subset of data and indexes for which the node is either a primary or backup one.

Apache Ignite Persistent Store has the following advantages over another databases (RDBMS, NoSQL, Hadoop) when the latter are used as a persistent layer for an Apache Ignite cluster:
* An ability to execute SQL queries over the data this is both in memory and on disk.
* No need to have all the data and indexes in memory. The Persistent Store allows storing a superset of data on disk and have only frequently used subsets in memory.
* Instantaneous cluster restarts. If the whole cluster goes down there is no need to warm up the memory preloading data from the Persistent Store. The cluster becomes fully operational once all the cluster nodes are interconnected with each other.
* Data and indexes are stored in a similar format both in memory and on disk that helps to avoid expensive transformations while the data sets are being moved or copied between the memory layers. 
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
That's it. Once the configuration parameter above is added to the cluster node configuration, the Persistent Store will be enabled and all the data as well as indexes will be stored both in memory and on disk cluster wide.
[block:callout]
{
  "type": "success",
  "title": "Persistent Store Root Path",
  "body": "By default all the data as well as write-ahead log files described below will be persisted under Apache Ignite working directory (`${IGNITE_HOME}/work`). Use `PersistentStoreConfiguration.setPersistentStorePath(...)` method to change the default directory. Also, as it will be shown below, there is a way to set a dedicated directory for the Write-Ahead Log file and its archives using special `PersistentStoreConfiguration` parameters."
}
[/block]

[block:api-header]
{
  "title": "Write-Ahead Log File"
}
[/block]
If [Page Memory](doc:page-memory) doesn't find a page in memory it will go to the Persistent Store to preload it from there. This can be easily achieved because all the pages are stored in separate partition files the pages belong to.

However, when a page is updated in memory the update is not directly written to a respective partition file in Persistent Store because it can affect performance dramatically. It's rather appended to the tail of an Apache Ignite node's write-ahead log (WAL) file that is maintained for all the deployed caches.

The purpose of the WAL file is to propagate updates to disk in the fastest way possible and provide a recovery mechanism for transactional updates written to the Persistent Store. Note, that every transactional update is uniquely defined cluster wide, which means that a cluster can always be recovered to the latest successfully committed transaction in case of a crash or restart.
[block:callout]
{
  "type": "success",
  "title": "More Details on WAL",
  "body": "Refer to WAL section on [Persistent Store Internal Design page](https://cwiki.apache.org/confluence/display/IGNITE/Persistent+Store+Internal+Design#PersistentStoreInternalDesign-Write-Ahead-Log) to learn more about WAL implementation in Apache Ignite."
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
TBD
[block:api-header]
{
  "title": "Transactional Guarantees"
}
[/block]
The Persistent Store is an ACID-compliant distributed store.

Every transactional update that comes to the store is appended to the WAL first. The update is uniquely defined with an ID. All this means that a cluster can always be recovered to the latest successfully committed transaction in case of a crash or restart.
[block:api-header]
{
  "title": "SQL"
}
[/block]
The Persistent Store allows using Apache Ignite as a distributed SQL database. The store is a fully ANSI-99 SQL compliant.

There is no need to have all the data in memory if you need to run SQL queries across the cluster. Apache Ignite is able to execute them over the data that is both in memory and on disk. 

Moreover, it's optional to preload data from the Persistent Store to the memory after a cluster's restart. Your applicant can run SQL queries as soon as the cluster is up and running.  
[block:api-header]
{
  "title": "Example"
}
[/block]
To see how the Persistent Store can be used in practice give a try to [this](https://github.com/apache/ignite/tree/ignite-5267/examples/src/main/java/org/apache/ignite/examples/persistentstore) example that is available on GitHub and delivered with every Apache Ignite distribution.