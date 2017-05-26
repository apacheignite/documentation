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
* An ability to execute SQL queries over the data and indexes that are both in-memory and on disk.
* No need to have all the data and indexes in-memory. The Persistent Store allows storing a superset of data on disk and have only frequently used data in-memory.
* Instantaneous cluster restarts. If the whole cluster goes down there is no need to warm up the memory preloading data from the Persistent Store. The cluster becomes fully operational once all the cluster nodes are interconnected with each other.
* Data and indexes are stored in a similar format both in-memory and on disk that helps to avoid expensive transformations while the data sets are being moved or copied between the memory layers. 
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

[block:api-header]
{
  "title": "Write-Ahead Log File"
}
[/block]

[block:api-header]
{
  "title": "Checkpointing"
}
[/block]

[block:api-header]
{
  "title": "Example"
}
[/block]