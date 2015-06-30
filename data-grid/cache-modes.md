---
title: "Cache Modes"
excerpt: "Setup different distribution models, backup copies, and near caches."
---
Ignite provides three different modes of cache operation: `LOCAL`, `REPLICATED`, and `PARTITIONED`. A cache mode is configured for each cache. Cache modes are defined in `CacheMode` enumeration. 
[block:api-header]
{
  "type": "basic",
  "title": "Local Mode"
}
[/block]
`LOCAL` mode is the most light weight mode of cache operation, as no data is distributed to other cache nodes. It is ideal for scenarios where data is either read-only, or can be periodically refreshed at some expiration frequency. It also works very well with `read-through` behavior where data is loaded from persistent storage on misses. Other than distribution, local caches still have all the features of a distributed cache, such as automatic data eviction, expiration, disk swapping, data querying, and transactions.
[block:api-header]
{
  "type": "basic",
  "title": "Replicated Mode"
}
[/block]
In `REPLICATED` mode all data is replicated to every node in the cluster. This cache mode provides the utmost availability of data as it is available on every node. However, in this mode every data update must be propagated to all other nodes which can have an impact on performance and scalability. 

As the same data is stored on all cluster nodes, the size of a replicated cache is limited by the amount of memory available on the node with the smallest amount of RAM. This mode is ideal for scenarios where cache reads are a lot more frequent than cache writes, and data sets are small. If your system does cache lookups over 80% of the time, then you should consider using `REPLICATED` cache mode.
[block:callout]
{
  "type": "success",
  "body": "Replicated caches should be used when data sets are small and updates are infrequent."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Partitioned Mode"
}
[/block]
`PARTITIONED` mode is the most scalable distributed cache mode. In this mode the overall data set is divided equally into partitions and all partitions are split equally between participating nodes, essentially creating one huge distributed in-memory store for caching data. This approach allows you to store as much data as can be fit in the total memory available across all nodes, hence allowing for multi-terabytes of data in cache memory across all cluster nodes. Essentially, the more nodes you have, the more data you can cache.

Unlike `REPLICATED` mode, where updates are expensive because every node in the cluster needs to be updated, with `PARTITIONED` mode, updates become cheap because only one primary node (and optionally 1 or more backup nodes) need to be updated for every key. However, reads become somewhat more expensive because only certain nodes have the data cached. 

In order to avoid extra data movement, it is important to always access the data exactly on the node that has that data cached. This approach is called *affinity colocation* and is strongly recommended when working with partitioned caches.
[block:callout]
{
  "type": "success",
  "body": "Partitioned caches are ideal when working with large data sets and updates are frequent.",
  "title": ""
}
[/block]
The picture below illustrates a simple view of a partitioned cache. Essentially we have key K1 assigned to Node1, K2 assigned to Node2, and K3 assigned to Node3. 
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/7pGSgxCVR3OZSHqYLdJv",
        "in_memory_data_grid.png",
        "500",
        "338",
        "#d64304",
        ""
      ]
    }
  ]
}
[/block]
See [configuration](#configuration) section below for an example on how to configure cache mode.
[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
Cache modes are configured for each cache by setting the `cacheMode` property of `CacheConfiguration` like so:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  \t...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n           \t<!-- Set a cache name. -->\n           \t<property name=\"name\" value=\"cacheName\"/>\n            \n          \t<!-- Set cache mode. -->\n    \t\t\t\t<property name=\"cacheMode\" value=\"PARTITIONED\"/>\n    \t\t\t\t... \n        </bean\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration(\"myCache\");\n\ncacheCfg.setCacheMode(CacheMode.PARTITIONED);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Atomic Write Order Mode"
}
[/block]
When using partitioned cache in `CacheAtomicityMode.ATOMIC` mode, one can configure atomic cache write order mode. Atomic write order mode determines which node will assign write version (sender or primary node) and is defined by `CacheAtomicWriteOrderMode` enumeration. There are 2 modes, `CLOCK` and `PRIMARY`. 

In `CLOCK` write order mode, write versions are assigned on a sender node. `CLOCK` mode is automatically turned on only when `CacheWriteSynchronizationMode.FULL_SYNC` is used, as it  generally leads to better performance since write requests to primary and backups nodes are sent at the same time. 

In `PRIMARY` write order mode, cache version is assigned only on primary node. In this mode the sender will only send write requests to primary nodes, which in turn will assign write version and forward them to backups.

Atomic write order mode can be configured via `atomicWriteOrderMode` property of `CacheConfiguration`. 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  \t...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n           \t<!-- Set a cache name. -->\n           \t<property name=\"name\" value=\"cacheName\"/>\n          \t\n          \t<!-- Atomic write order mode. -->\n    \t\t\t\t<property name=\"atomicWriteOrderMode\" value=\"PRIMARY\"/>\n    \t\t\t\t... \n        </bean\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\ncacheCfg.setAtomicWriteOrderMode(CacheAtomicWriteOrderMode.CLOCK);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "For more information on `ATOMIC` mode, refer to [Transactions](/docs/transactions) section."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  \t...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n           \t<!-- Set a cache name. -->\n           \t<property name=\"name\" value=\"cacheName\"/>\n          \n          \t<!-- Set cache mode. -->\n    \t\t\t\t<property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          \t\n          \t<!-- Number of backup nodes. -->\n    \t\t\t\t<property name=\"backups\" value=\"1\"/>\n    \t\t\t\t... \n        </bean\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\ncacheCfg.setCacheMode(CacheMode.PARTITIONED);\n\ncacheCfg.setBackups(1);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]