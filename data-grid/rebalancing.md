* [Overview](#overview)
* [Rebalance Modes](#rebalance-modes)
* [Rebalance Thread Pool Tuning](#rebalance-thread-pool-tuning)
* [Rebalance Message Throttling](#rebalance-message-throttling)
* [Configuration](#configuration)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
When a new node joins topology, existing nodes relinquish primary or back up ownership of some keys to the new node so that keys remain equally balanced across the grid at all times.

If the new node becomes a primary or backup for some partition, it will fetch data from previous primary node for that partition or from one of the backup nodes for that partition. Once a partition is fully loaded to the new node, it will be marked obsolete on the old node and will be eventually evicted after all current transactions on that node are finished. Hence, for some short period of time, after topology changes, there can be a case when a cache will have more backup copies for a key than configured. However once rebalancing completes, extra backup copies will be removed from node caches.
[block:api-header]
{
  "type": "basic",
  "title": "Rebalance Modes"
}
[/block]
Following rebalance modes are defined in `CacheRebalanceMode` enum.
[block:parameters]
{
  "data": {
    "0-0": "`SYNC`",
    "h-0": "CacheRebalanceMode",
    "h-1": "Description",
    "0-1": "Synchronous rebalancing mode. Distributed caches will not start until all necessary data is loaded from other available grid nodes. This means that any call to cache public API will be blocked until rebalancing is finished.",
    "1-1": "Asynchronous rebalancing mode. Distributed caches will start immediately and will load all necessary data from other available grid nodes in the background.",
    "1-0": "`ASYNC`",
    "2-1": "In this mode no rebalancing will take place which means that caches will be either loaded on demand from persistent store whenever data is accessed, or will be populated explicitly.",
    "2-0": "`NONE`"
  },
  "cols": 2,
  "rows": 3
}
[/block]
By default, `ASYNC` rebalance mode is enabled. To use another mode, you can set the `rebalanceMode` property of `CacheConfiguration`, like so:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">          \t\t\n          \t<!-- Set synchronous rebalancing. -->\n    \t\t\t\t<property name=\"rebalanceMode\" value=\"SYNC\"/>\n            ... \n        </bean\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setRebalanceMode(CacheRebalanceMode.SYNC);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Rebalance Thread Pool Tuning"
}
[/block]
`IgniteConfiguration` provides `setRebalanceThreadPoolSize` method that allows to set a number of threads that will be taken from the Ignite's system thread pool and used for rebalancing needs. A system thread is taken from the pool every time a node needs to send a batch of data to a remote node, that maybe primary or backup for a partition, or needs to process a batch that came from the opposite direction. The thread is relinquished every time the batch is sent or received and processed. 

By default, only one thread is used for rebalancing needs. Basically it means that at a particular point of time only one thread will be used to transfer batches from one node to another, or to process batches coming from the remote side. As an example, if the cluster has two nodes and a single cache, then all the cache's partitions will be re-balanced sequentially, one by one. If the cluster has two nodes and two different caches, then these caches will be re-balanced in parallel, but at a particular point of time only batches that belong to a particular cache will be processed as explained above.
[block:callout]
{
  "type": "success",
  "body": "Number of partitions per cache doesn't affect rebalancing performance. What makes sense is the total amount of data, rebalance thread pool size and other parameters listed in the sections below."
}
[/block]
Depending on the number of caches in the system and amount of data stored in the caches, if the rebalance thread pool's size is equal to `1`, it can take a significant amount of time before all of the data is re-balanced to a node. To speed up the preloading process, you can increase  `IgniteConfiguration.setRebalanceThreadPoolSize` to the value that is applicable for your case.

 Let's imagine that `IgniteConfiguration.setRebalanceThreadPoolSize` is set to `4` and considering the examples provided above, the rebalancing behavior will be the following - 
  * If the cluster has two nodes and a single cache, then the cache's partitions will be logically put in 4 different groups which will be rebalanced in parallel by one of the 4 threads. Partitions that belong to a particular group will be rebalanced sequentially, one by one.
  * If the cluster has two nodes and two different caches, then partitions of every cache will be logically put in 4 different groups (each cache will have its own 4 groups giving 8 group in total) and the groups will be re-balanced in parallel by four different threads. However at a particular point of time only batches that belong to a group (8 in total) will be processed as explained above. 
[block:callout]
{
  "type": "warning",
  "body": "System thread pool is widely used internally by all the cache related operations (put, get, etc.), SQL engine and other modules. Setting `IgniteConfiguration.setRebalanceThreadPoolSize` to a big value may significantly increase rebalancing performance, affecting your application throughput."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Rebalance Message Throttling"
}
[/block]
When re-balancer transfers data from one node to another, it splits the whole data set into batches and sends each batch in a separate message. If your data sets are large and there are a lot of messages to send, the CPU or network can get over-consumed. In this case it can be reasonable to wait between rebalance messages so that negative performance impact caused by rebalancing process is minimized. This time interval is controlled by `rebalanceThrottle` configuration property of  `CacheConfiguration`. Its default value is 0, which means that there will be no pauses between messages. Note that size of a single message can be also customized by `rebalanceBatchSize` configuration property (default size is 512K).

For example, if you want rebalancer to send 2MB of data per message with 100 ms throttle interval, you should provide the following configuration: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">          \t\t\n          \t<!-- Set batch size. -->\n    \t\t\t\t<property name=\"rebalanceBatchSize\" value=\"#{2 * 1024 * 1024}\"/>\n \n    \t\t\t\t<!-- Set throttle interval. -->\n    \t\t\t\t<property name=\"rebalanceThrottle\" value=\"100\"/>\n            ... \n        </bean\n    </property>\n</bean> ",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setRebalanceBatchSize(2 * 1024 * 1024);\n            \ncacheCfg.setRebalanceThrottle(100);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
Cache rebalancing behavior can be customized by optionally setting the following configuration properties:

CacheConfiguration
[block:parameters]
{
  "data": {
    "h-0": "Setter Method",
    "h-1": "Description",
    "h-2": "Default",
    "0-0": "`setRebalanceMode`",
    "0-1": "Rebalance mode for distributed cache. See Rebalance Modes section for details.",
    "1-0": "`setRebalanceDelay`",
    "1-1": "Delay in milliseconds upon a node joining or leaving topology (or crash) after which rebalancing should be started automatically. Rebalancing should be delayed if you plan to restart nodes after they leave topology, or if you plan to start multiple nodes at once or one after another and don't want to repartition and rebalance until all nodes are started.",
    "2-0": "`setRebalanceBatchSize`",
    "2-1": "Size (in bytes) to be loaded within a single rebalance message. Rebalancing algorithm will split total data set on every node into multiple batches prior to sending data.",
    "3-0": "`setRebalanceThrottle`",
    "3-1": "See `Rebalance Message Throttling` section above for details.",
    "4-0": "`setRebalanceOrder`",
    "4-1": "Order in which rebalancing should be done. Rebalance order can be set to non-zero value for caches with SYNC or ASYNC rebalance modes only. Rebalancing for caches with smaller rebalance order will be completed first. By default, rebalancing is not ordered.",
    "0-2": "`ASYNC`",
    "1-2": "0 (no delay)",
    "2-2": "512K",
    "3-2": "0 (throttling disabled)",
    "4-2": "0",
    "5-0": "`setRebalanceBatchesPrefetchCount`",
    "5-1": "To gain better rebalancing performance, Supplier node can provide more than one batch at rebalancing start and provide one new to each next demand request.\nSets number of batches generated by supply node at rebalancing start.",
    "5-2": "2",
    "6-0": "`setRebalanceTimeout`",
    "6-1": "Timeout for pending rebalancing messages that are being exchanged between the nodes",
    "6-2": "10 seconds"
  },
  "cols": 3,
  "rows": 7
}
[/block]
IgniteConfiguration
[block:parameters]
{
  "data": {
    "h-0": "Setter Method",
    "h-1": "Description",
    "h-2": "Default",
    "0-2": "1",
    "0-0": "`setRebalanceThreadPoolSize`",
    "0-1": "See `Rebalance Thread Pool Tuning` section above for details."
  },
  "cols": 3,
  "rows": 1
}
[/block]