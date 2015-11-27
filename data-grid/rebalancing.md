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
    "1-0": "`setRebalancePartitionedDelay`",
    "1-1": "Rebalancing delay in milliseconds. See Delayed And Manual Rebalancing section for details.",
    "2-0": "`setRebalanceBatchSize`",
    "2-1": "Size (in bytes) to be loaded within a single rebalance message. Rebalancing algorithm will split total data set on every node into multiple batches prior to sending data.",
    "3-0": "`setRebalanceThrottle`",
    "3-1": "Time in milliseconds to wait between rebalancing messages to avoid overloading of CPU or network. When rebalancing large data sets, the CPU or        network can get over-  consumed with rebalance messages, which consecutively may slow down the application performance. This parameter helps tune the amount of time to wait between rebalance messages to make sure that rebalancing process does not have any negative performance impact. Note that application will continue to work properly while rebalancing is still in progress.",
    "4-0": "`setRebalanceOrder`",
    "5-0": "`setRebalanceTimeout`",
    "4-1": "Order in which rebalancing should be done. Rebalance order can be set to non-zero value for caches with SYNC or ASYNC rebalance modes only. Rebalancing for caches with smaller rebalance order will be completed first. By default, rebalancing is not ordered.",
    "5-1": "Rebalance timeout (ms).",
    "0-2": "`ASYNC`",
    "1-2": "0 (no delay)",
    "2-2": "512K",
    "3-2": "0 (throttling disabled)",
    "4-2": "0",
    "5-2": "10000",
    "6-0": "`setRebalanceBatchesPrefetchCount",
    "6-1": "To gain better rebalancing performance Supplier node can provide more than one batch at rebalancing start and provide one new to each next demand request.\nSets number of batches generated by supply node at rebalancing start.",
    "6-2": "2"
  },
  "cols": 3,
  "rows": 7
}
[/block]
IgniteConfiguration
[block:parameters]
{
  "data": {
    "0-0": "`setRebalanceThreadPoolSize`",
    "0-1": "Max count of threads can be used at rebalancing",
    "0-2": "1 (has minimal impact on the operation of the grid)"
  },
  "cols": 3,
  "rows": 1
}
[/block]