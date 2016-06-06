Ignite in-memory data grid has been built from the ground up with a notion of horizontal scale and ability to add nodes on demand in real-time; it has been designed to linearly scale to hundreds of nodes with strong semantics for data locality and affinity data routing to reduce redundant data noise.

Ignite data grid is an `in-memory distributed key-value store` which can be viewed as a distributed partitioned hash map, with every cluster node owning a portion of the overall data. This way the more cluster nodes we add, the more data we can cache.

Unlike other key-value stores, Ignite determines data locality using a pluggable hashing algorithm. Every client can determine which node a key belongs to by plugging it into a hashing function, without a need for any special mapping servers or name nodes.

Ignite data grid supports local, replicated, and partitioned data sets and allows to freely cross query between these data sets using standard SQL syntax. Ignite supports standard SQL for querying in-memory data including support for distributed SQL joins. 

Ignite data grid is lightning fast and is one of the fastest implementations of transactional or atomic data in a  cluster today.
[block:callout]
{
  "type": "success",
  "title": "Data Consistency",
  "body": "As long as your cluster is alive, Ignite will guarantee that the data between different cluster nodes will always remain consistent regardless of crashes or topology changes."
}
[/block]

[block:callout]
{
  "type": "success",
  "title": "JCache (JSR 107)",
  "body": "Ignite Data Grid implements [JCache](doc:jcache) (JSR 107) specification."
}
[/block]

[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/ZBWQwPXbQmyq6RRUyWfm",
        "in-memory-data-grid-1.jpg",
        "500",
        "338",
        "#e8893c",
        ""
      ]
    }
  ]
}
[/block]
##Features
* [JCache and Beyond](doc:jcache) 
* [Cache Modes](doc:cache-modes) 
* [Primary & Backup Copies](doc:primary-and-backup-copies) 
* [Near Caches](doc:near-caches) 
* [Cache Queries](doc:cache-queries) 
* [SQL Queries](doc:sql-queries) 
* [Continuous Queries](doc:continuous-queries) 
* [ ACID Transactions](doc:transactions) 
* [Locks](doc:distributed-locks) 
* [Off-Heap Memory](doc:off-heap-memory) 
* [Affinity Collocation](doc:affinity-collocation) 
* [Persistent Store](doc:persistent-store) 
* [Automatic Persistence](doc:automatic-persistence) 
* [Data Loading](doc:data-loading) 
* [Eviction Policies](doc:evictions) 
* [Expiry Policies](doc:expiry-policies) 
* [Data Rebalancing](doc:rebalancing) 
* [Web Session Clustering](doc:web-session-clustering) 
* [Hibernate L2 Cache](doc:hibernate-l2-cache) 
* [JDBC Driver](doc:jdbc-driver) 
* [Spring Caching](doc:spring-caching) 
* [Topology Validation](doc:topology-validation)