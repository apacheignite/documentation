* [Memory Metrics](#memory-metrics)
* [Cache Metrics](#cache-metrics)
[block:api-header]
{
  "title": "Memory Metrics"
}
[/block]
Apache Ignite’s [page memory](doc:page-memory) can be analyzed via several parameters exposed through `MemoryMetrics` interface and JMX bean. Having access to the memory metrics can help track the overall memory utilization, measure its performance, and execute required optimizations if a bottleneck is spotted.

`MemoryMetrics` is the main entry point that provides page memory-related metrics of a specific Apache Ignite node. Since there are can be several memory regions configured on a node, the metrics for every region are collected and obtained separately using this API.

Currently, `MemoryMetrics` interface supports the following methods:
[block:parameters]
{
  "data": {
    "h-0": "Method Name",
    "h-1": "Description",
    "0-0": "`getName()`",
    "0-1": "Returns a name of a memory region the metrics belong to.",
    "1-0": "`getTotalAllocatedPages()`",
    "1-1": "Gets a total number of allocated pages in the memory region.",
    "2-0": "`getAllocationRate()`",
    "2-1": "Gets pages allocation rate in this memory region.",
    "3-0": "`getEvictionRate()`",
    "3-1": "Gets pages eviction rate in the given memory region.",
    "4-0": "`getLargeEntriesPagesPercentage()`",
    "4-1": "Gets a percentage of pages that are fully occupied by large entries that go beyond page size. The large entities are split into fragments in a way so that each fragment can fit into a single page.",
    "5-0": "`getPagesFillFactor()`",
    "5-1": "Gets a percentage of space that is still free and can be filled in."
  },
  "cols": 2,
  "rows": 6
}
[/block]
Call `Ignite.memoryMetrics()` method to get the latest metrics snapshot and iterate over it, as shown in the example below:
[block:code]
{
  "codes": [
    {
      "code": "// Getting the metrics of all the memory regions defined on the node.\nCollection<MemoryMetrics> regionsMetrics = ignite.memoryMetrics();\n\n// Printing out some of the metrics' probes for all the regions.\nfor (MemoryMetrics metrics : regionsMetrics) {\n\tSystem.out.println(\">>> Memory Region Name: \" + metrics.getName());\n  System.out.println(\"    >>> Allocation Rate: \" + metrics.getAllocationRate());\n  System.out.println(\"    >>> Fill Factor: \" + metrics.getPagesFillFactor());\n\n  System.out.println();\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "Enabling Metrics Gathering",
  "body": "Memory metrics collection is not a free operation and might affect performance of an application. This is the reason why the memory metrics gathering is turned off by default. \n\nTo turn the metrics on use one of the following approaches:\n* Set `MemoryPolicyConfiguration.setMetricsEnabled(boolean)` to `true` for every memory region you want to gather the metrics for.\n* Use `MemoryMetricsMXBean.enableMetrics()` method exposed by a special JMX bean described below in the documentation."
}
[/block]

Alternatively, the page memory state can be observed with `MemoryMetricsMXBean` interface. You can connect to the bean from any JMX-compliant tool or API.

The JMX bean exposes the same set of metrics `MemoryMetrics` interface has and, at the same time, offers some new listed below. Some of these bean-specific methods will be added to `MemoryMetrics` interface in the future releases:
[block:parameters]
{
  "data": {
    "0-0": "`getInitialSize()`",
    "0-1": "Gets initial memory region size defined by its memory policy configuration.",
    "1-0": "`getMaxSize()`",
    "1-1": "Gets maximum memory region size defined by its memory policy configuration.",
    "2-0": "`getSwapFilePath()`",
    "2-1": "Gets a path to the memory-mapped files, if any, a memory region will be mapped to.",
    "3-0": "`enableMetrics()`",
    "3-1": "Enables memory metrics collection on an Apache Ignite node for a specific memory region.",
    "4-0": "`disableMetrics()`",
    "4-1": "Disables memory metrics collection on an Apache Ignite node for a specific memory region.",
    "5-0": "`rateTimeInterval(int)`",
    "5-1": "Sets a time interval for pages allocation and eviction rates monitoring purposes. For instance, after setting the interval to 60 seconds, subsequent calls to `getAllocationRate()` will return average allocation rate (pages per second) for the last minute.",
    "6-0": "`subIntervals(int)`",
    "6-1": "Sets a number of sub-intervals the whole `rateTimeInterval(int)` will be split into to calculate pages allocation and eviction rates (5 by default). Setting this parameter to a bigger value will result in the more precise calculation."
  },
  "cols": 2,
  "rows": 7
}
[/block]

[block:api-header]
{
  "title": "Cache Metrics"
}
[/block]
In addition to the low-level memory related metrics explained above, Apache Ignite allows keeping an eye on distributed caches specific statistics available via `CacheMetrics` interface.

The interface has a variety of metrics such as a total number of put and get operations processed by a cache, average put or get time, a total number of evictions, current write-behind cache store buffer size and more. Refer to `CacheMetrics` javadoc to see a complete list of all the metrics available.

There are several ways how you can the latest metrics snapshot of a specific cache:
* `IgniteCache.metrics()` - gets metrics snapshot of the whole cluster where the cache is deployed.
* `IgniteCache.metrics(ClusterGroup grp)` - gets metrics snapshot for Apache Ignite nodes that belong to a passed cluster group.
* `IgniteCache.localMetrics()` - gets local node's metrics snapshot for the cache.

Alternatively, you can get an access to the cache metrics via `CacheMetricsMXBean` interface. You can connect to the bean from any JMX-compliant tool or API. If you need to work with the bean from your application use `IgniteCache.mxBean()` or `IgniteCache.localMxBean()` to get a bean reference.
[block:callout]
{
  "type": "warning",
  "title": "Enabling Cache Metrics",
  "body": "To enable cache metrics gathering set `CacheConfiguration.setStatisticsEnabled(boolean)` to `true` for every cache you want to collect metrics for."
}
[/block]