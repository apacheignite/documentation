Ignite In-Memory Data Grid performance and throughput vastly depends on the features and the settings you use. In almost any use case the cache performance can be optimized by simply tweaking the cache configuration.
[block:api-header]
{
  "type": "basic",
  "title": "Disable Internal Events Notification"
}
[/block]
Ignite has rich event system to notify users about various events, including cache modification, eviction, compaction, topology changes, and a lot more. Since thousands of events per second are generated, it creates an additional load on the system. This can lead to significant performance degradation. Therefore, it is highly recommended to enable only those events that your application logic requires. By default, event notifications are disabled.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ... \n    <!-- Enable only some events and leave other ones disabled. -->\n    <property name=\"includeEventTypes\">\n        <list>\n            <util:constant static-field=\"org.apache.ignite.events.EventType.EVT_TASK_STARTED\"/>\n            <util:constant static-field=\"org.apache.ignite.events.EventType.EVT_TASK_FINISHED\"/>\n            <util:constant static-field=\"org.apache.ignite.events.EventType.EVT_TASK_FAILED\"/>\n        </list>\n    </property>\n    ...\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Tune Cache Start Size"
}
[/block]
In terms of size and capacity, Ignite's internal cache map acts exactly like a normal Java HashMap: it has some initial capacity (which is pretty small by default), which doubles as data arrives. The process of internal cache map resizing is CPU-intensive and time-consuming, and if you load a huge dataset into cache (which is a normal use case), the map will have to resize a lot of times. To avoid that, you can specify the initial cache map capacity, comparable to the expected size of your dataset. This will save a lot of CPU resources during the load time, because the map won't have to resize. For example, if you expect to load 100 million entries into cache, you can use the following configuration:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n            ...\n            <!-- Set initial cache capacity to ~ 100M. -->\n            <property name=\"startSize\" value=\"#{100 * 1024 * 1024}\"/> \n            ...\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
The above configuration will save you from log₂(10⁸) − log₂(1024) ≈ 16 cache map resizes (1024 is an initial map capacity by default). Remember, that each subsequent resize will be on average 2 times longer than the previous one.
[block:api-header]
{
  "type": "basic",
  "title": "Turn Off Backups"
}
[/block]
If you use `PARTITIONED` cache, and the data loss is not critical for you (for example, when you have a backing cache store), consider disabling backups for `PARTITIONED` cache. When backups are enabled, the cache engine has to maintain a remote copy of each entry, which requires network exchange and is time-consuming. To disable backups, use the following configuration:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n            ...\n            <!-- Set cache mode. -->\n            <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n            <!-- Set number of backups to 0-->\n            <property name=\"backups\" value=\"0\"/>\n            ...\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "If you don't have backups enabled for `PARTITIONED` cache, you will loose all entries cached on a failed node. It may be acceptable for caching temporary data or data that can be otherwise recreated. Make sure that such data loss is not critical for application before disabling backups.",
  "title": "Possible Data Loss"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Tune Off-Heap Memory"
}
[/block]
If you plan to allocate large amounts of memory to your JVM for data caching (usually more than 10GB of memory), then your application will most likely suffer from prolonged lock-the-world GC pauses which can significantly hurt latencies. To avoid GC pauses use off-heap memory to cache data - essentially your data is still cached in memory, but JVM does not know about it and GC is not affected. To enable off-heap storage with unlimited size, use the following configuration:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n            ...\n            <!-- Enable off-heap storage with unlimited size. -->\n            <property name=\"offHeapMaxMemory\" value=\"0\"/> \n            ...\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Disable Swap Storage"
}
[/block]
Swap storage is disabled by default. However, in your configuration it might be enabled. If it is, keep in mind that using swap storage can significantly hurt performance. To disable swap storage explicitly, use the following configuration:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n            ...\n            <!-- Disable swap. -->\n            <property name=\"swapEnabled\" value=\"false\"/> \n            ...\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Disable Peer Class Loading"
}
[/block]
While peer class loading is very convenient in development, it does carry a certain overhead and should be turned off in production. To disable peer class loading, use the following configuration snippet:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n            ...\n            <!-- Explicitly disable peer class loading. -->\n            <property name=\"peerClassLoadingEnabled\" value=\"false\"/> \n            ...\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Tune Eviction Policy"
}
[/block]
Evictions are disabled by default. If you do need to use evictions to make sure that data in cache does not overgrow beyond allowed memory limits, consider choosing the proper eviction policy.  An example of setting the LRU eviction policy with maximum size of 100000 entries is shown below:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n    ...\n    <property name=\"evictionPolicy\">\n        <!-- LRU eviction policy. -->\n        <bean class=\"org.apache.ignite.cache.eviction.lru.LruEvictionPolicy\">\n            <!-- Set the maximum cache size to 1 million (default is 100,000). -->\n            <property name=\"maxSize\" value=\"1000000\"/>\n        </bean>\n    </property>\n    ...\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
Regardless of which eviction policy you use, cache performance will depend on the maximum amount of entries in cache allowed by eviction policy - if cache size overgrows this limit, the evictions start to occur.
[block:api-header]
{
  "type": "basic",
  "title": "Tune Cache Data Rebalancing"
}
[/block]
When a new node joins topology, existing nodes relinquish primary or back up ownership of some keys to the new node so that keys remain equally balanced across the grid at all times. This may require additional resources and hit cache performance. To tackle this possible problem, consider tweaking the following parameters:
  * Configure rebalance batch size, appropriate for your network. Default is 512KB which means that by default rebalance messages will be about 512KB. However, you may need to set this value to be higher or lower based on your network performance.
  * Configure rebalance throttling to unload the CPU. If your data sets are large and there are a lot of messages to send, the CPU or network can get over-consumed, which consecutively may slow down the application performance. In this case you should enable data rebalance throttling which helps tune the amount of time to wait between rebalance messages to make sure that rebalancing process does not have any negative performance impact. Note that application will continue to work properly while rebalancing is still in progress.
  * Configure rebalance thread pool size. As opposite to previous point, sometimes you may need to make rebalancing faster by engaging more CPU cores. This can be done by increasing the number of threads in rebalance thread pool (by default, there are only 2 threads in pool).

Below is an example of setting all of the above parameters in cache configuration:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">             \n            <!-- Set rebalance batch size to 1 MB. -->\n            <property name=\"rebalanceBatchSize\" value=\"#{1024 * 1024}\"/>\n \n            <!-- Explicitly disable rebalance throttling. -->\n            <property name=\"rebalanceThrottle\" value=\"0\"/>\n \n            <!-- Set 4 threads for rebalancing. -->\n            <property name=\"rebalanceThreadPoolSize\" value=\"4\"/>\n            ... \n        </bean\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configure Thread Pools"
}
[/block]
By default, Ignite has it's main thread pool size set to the 2 times the available CPU count. In most cases keeping 2 threads per core will result in faster application performance, since there will be less context switching and CPU caches will work better. However, if you are expecting that your jobs will block for I/O or any other reason, it may make sense to increase the thread pool size. Below is an examples of how you configure thread pools:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ... \n    <!-- Configure internal thread pool. -->\n    <property name=\"publicThreadPoolSize\" value=\"64\"/>\n    \n    <!-- Configure system thread pool. -->\n    <property name=\"systemThreadPoolSize\" value=\"32\"/>\n    ...\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Use Collocated Computations"
}
[/block]
Ignite enables you to execute MapReduce computations in memory. However, most computations usually work on some data which is cached on remote grid nodes. Loading that data from remote nodes is very expensive in most cases and it is a lot more cheaper to send the computation to the node where the data is. The easiest way to do it is to use `IgniteCompute.affinityRun()` method or `@CacheAffinityMapped` annotation. There are other ways, including `Affinity.mapKeysToNodes()` methods. The topic of collocated computations is covered in much detail in [Affinity Collocation](doc:affinity-collocation), which contains proper code examples.
[block:api-header]
{
  "type": "basic",
  "title": "Use Data Streamer"
}
[/block]
If you need to upload lots of data into cache, use `IgniteDataStreamer` to do it. Data streamer will properly batch the updates prior to sending them to remote nodes and will properly control number of parallel operations taking place on each node to avoid thrashing. Generally it provides performance of 10x than doing a bunch of single-threaded updates. See [Data Loading](doc:data-loading) section for more detailed description and examples.
[block:api-header]
{
  "type": "basic",
  "title": "Batch Up Your Messages"
}
[/block]
If you can send 10 bigger jobs instead of 100 smaller jobs, you should always choose to send bigger jobs. This will reduce the amount of jobs going across the network and may significantly improve performance. The same regards cache entries - always try to use API methods, that take collections of keys or values, instead of passing them one-by-one.