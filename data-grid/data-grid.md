Ignite in-memory data grid has been built from the ground up with a notion of horizontal scale and ability to add nodes on demand in real-time; it has been designed to linearly scale to hundreds of nodes with strong semantics for data locality and affinity data routing to reduce redundant data noise.

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
  * Distributed In-Memory Caching
  * Lightning Fast Performance
  * Elastic Scalability
  * Distributed In-Memory Transactions
  * Web Session Clustering
  * Hibernate L2 Cache Integration
  * Tiered Off-Heap Storage
  * Distributed ANSI-99 SQL Queries with support for Joins
[block:api-header]
{
  "type": "basic",
  "title": "IgniteCache"
}
[/block]
`IgniteCache` interface is a gateway into Ignite cache implementation and provides methods for storing and retrieving data, executing queries, including SQL, iterating and scanning, etc.

##JCache
`IgniteCache` interface extends `javax.cache.Cache` interface from JCache specification and adds additional functionality to it, mainly having to do with local vs. distributed operations, queries, metrics, etc.

You can obtain an instance of `IgniteCache` as follows:
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\n// Obtain instance of cache named \"myCache\".\n// Note that different caches may have different generics.\nIgniteCache<Integer, String> cache = ignite.cache(\"myCache\");",
      "language": "java"
    }
  ]
}
[/block]
You can also create an instance of the cache on the fly, in which case Ignite will create and deploy the cache across all server cluster members.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nCacheConfiguration cfg = new CacheConfiguration();\n\ncfg.setName(\"myCache\");\ncfg.setAtomicityMode(TRANSACTIONAL);\n\n// Create cache with given name, if it does not exist.\nIgniteCache<Integer, String> cache = ignite.getOrCreateCache(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "XML Configuration",
  "body": "All caches defined in Ignite Spring XML configuration on any cluster member will also be automatically created and deployed on all the cluster servers (no need to specify the same configuration on each cluster member)."
}
[/block]