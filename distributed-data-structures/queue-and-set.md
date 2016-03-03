Ignite In-Memory Data Fabric, in addition to providing standard key-value map-like storage, also provides an implementation of a fast Distributed Blocking Queue and Distributed Set.

`IgniteQueue` and `IgniteSet`, an implementation of `java.util.concurrent.BlockingQueue` and `java.util.Set` interface respectively,  also support all operations from `java.util.Collection` interface. Both, queue and set can be created in either collocated or non-collocated mode.

Below is an example of how to create a distributed queue and set.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nIgniteQueue<String> queue = ignite.queue(\n    \"queueName\", // Queue name.\n    0,          // Queue capacity. 0 for unbounded queue.\n    null         // Collection configuration.\n);",
      "language": "java",
      "name": "Queue"
    },
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nIgniteSet<String> set = ignite.set(\n    \"setName\", // Set name.\n    null       // Collection configuration.\n);",
      "language": "java",
      "name": "Set"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Collocated vs. Non-Collocated Mode"
}
[/block]
If you plan to create just a few queues or sets containing lots of data, then you would create them in non-collocated mode. This will make sure that about equal portion of each queue or set will be stored on each cluster node. On the other hand, if you plan to have many queues or sets, relatively small in size (compared to the whole cache), then you would most likely create them in collocated mode. In this mode all queue or set elements will be stored on the same cluster node, but about equal amount of queues/sets will be assigned to every node.
A collocated queue and set can be created by setting the `collocated` property of `CollectionConfiguration`, like so:
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nCollectionConfiguration colCfg = new CollectionConfiguration();\n\ncolCfg.setCollocated(true); \n\n// Create collocated queue.\nIgniteQueue<String> queue = ignite.queue(\"queueName\", 0, colCfg);",
      "language": "java",
      "name": "Queue"
    },
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nCollectionConfiguration colCfg = new CollectionConfiguration();\n\ncolCfg.setCollocated(true); \n\n// Create collocated set.\nIgniteSet<String> set = ignite.set(\"setName\", colCfg);",
      "language": "java",
      "name": "Set"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "",
  "body": "Non-collocated mode only makes sense for and is only supported for `PARTITIONED` caches."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Cache Queues and Load Balancing"
}
[/block]
Given that elements will remain in the queue until someone takes them, and that no two nodes should ever receive the same element from the queue, cache queues can be used as an alternate work distribution and load balancing approach within Ignite. 

For example, you could simply add computations, such as instances of `IgniteRunnable` to a queue, and have threads on remote nodes call `IgniteQueue.take()`  method which will block if queue is empty. Once the `take()` method will return a job, a thread will process it and call `take()` again to get the next job. Given this approach, threads on remote nodes will only start working on the next job when they have completed the previous one, hence creating ideally balanced system where every node only takes the number of jobs it can process, and not more.
[block:api-header]
{
  "type": "basic",
  "title": "Collection Configuration"
}
[/block]
Ignite collections can be in configured in API via `CollectionConfiguration` (see above examples). The following configuration parameters can be used:
[block:parameters]
{
  "data": {
    "h-0": "Setter Method",
    "0-0": "`setCollocated(boolean)`",
    "h-1": "Description",
    "h-2": "Default",
    "0-2": "`false`",
    "0-1": "Sets collocation mode.",
    "1-0": "`setCacheMode(CacheMode)`",
    "2-0": "`setAtomicityMode(CacheAtomicityMode)`",
    "3-0": "`setMemoryMode(CacheMemoryMode)`",
    "4-0": "`setOffHeapMaxMemory(long)`",
    "5-0": "`setBackups(int)`",
    "6-0": "`setNodeFilter(IgnitePredicate<ClusterNode>)`",
    "1-1": "Sets underlying cache mode (`PARTITIONED`, `REPLICATED` or `LOCAL`).",
    "2-1": "Sets underlying cache atomicity mode (`ATOMIC` or `TRANSACTIONAL`).",
    "3-1": "Sets underlying cache memory mode (`ONHEAP_TIERED`, `OFFHEAP_TIERED` or `OFFHEAP_VALUES`).",
    "4-1": "Sets offheap maximum memory size.",
    "5-1": "Sets number of backups.",
    "6-1": "Sets optional predicate specifying on which nodes entries should be stored.",
    "1-2": "`PARTITIONED`",
    "2-2": "`ATOMIC`",
    "3-2": "`ONHEAP_TIERED`",
    "4-2": "`0` (unlimited)",
    "5-2": "`0`"
  },
  "cols": 3,
  "rows": 7
}
[/block]