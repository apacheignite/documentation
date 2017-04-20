* [Overview](#overview)
* [Page Memory Eviction Modes](#page-memory-eviction-modes)
* [On-heap Cache Eviction Policies](#on-heap-cache-eviction-policies)
 * [Least Recently Used](#least-recently-used-lru)
 * [First In First Out](#first-in-first-out-fifo)
 * [Sorted](#sorted)
 * [Random](#random)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
There are two distinct data eviction concepts that are supported in Apache Ignite and used for different purposes - page-based eviction for the off-heap [page memory](doc:page-memory) and cache entries based eviction for the optional [page memory's on-heap cache](https://apacheignite.readme.io/docs/page-memory#section-on-heap-caching).

The page-based eviction is configured via page memory policies and covered in the next section below while the entries based eviction can be enabled with eviction policies that are explained in [on-heap cache eviction policies](#on-heap-cache-eviction-policies) part of the documentation.
[block:api-header]
{
  "title": "Page Memory Eviction Modes"
}
[/block]
[Page Memory](doc:page-memory) consists of one or more memory pools configured by `MemoryPolicyConfigurations`. By default, a pool constantly grows in size until its maximum size is reached.

To avoid possible pool exhaustion you might need to set one of data page eviction modes via  `MemoryPolicyConfiguration.setPageEvictionMode(...)` configuration parameter. Basically, the eviction modes track data pages usage and evict some of them according to a mode's implementation.

## Random-LRU Mode

To enable Random-LRU eviction algorithm pass `DataPageEvictionMode.RANDOM_LRU` value to a respective `MemoryPolicyConfiguration` as it's shown in the example below: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.MemoryConfiguration\">\n  <!-- Defining additional memory poolicies. -->\n  <property name=\"memoryPolicies\">\n    <list>\n      <!--\n          Defining a policy for 10 GB memory pool with RANDOM_LRU eviction.\n      -->\n      <bean class=\"org.apache.ignite.configuration.MemoryPolicyConfiguration\">\n        <property name=\"name\" value=\"20GB_Pool_Eviction\"/>\n        <!-- Total size of 20 GB. -->\n        <property name=\"size\" value=\"#{20L * 1024 * 1024 * 1024}\"/>\n        <!-- Enabling RANDOM_LRU eviction. -->\n        <property name=\"pageEvictionMode\" value=\"RANDOM_2_LRU\"/>\n      </bean>\n    </list>\n    ...\n  </property>\n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "// Defining additional memory poolicies.\nMemoryConfiguration memCfg = new MemoryConfiguration();\n\n// Defining a policy for 10 GB memory pool with RANDOM_LRU eviction.\nMemoryPolicyConfiguration memPlc = new MemoryPolicyConfiguration();\n\nmemPlc.setName(\"20GB_Pool_Eviction\");\n\n// Total size of 20 GB.\nmemPlc.setSize(20L * 1024 * 1024 * 1024);\n\n// Enabling RANDOM_LRU eviction.\nmemPlc.setPageEvictionMode(DataPageEvictionMode.RANDOM_LRU);\n        \n// Setting the new memory policy.\nmemCfg.setMemoryPolicies(memPlc);",
      "language": "java"
    }
  ]
}
[/block]
Random-LRU algorithm works this way:
* Once a memory pool defined by a memory policy is configured, an off-heap array is allocated to track 'last usage' timestamp for every individual data page.
* When a data page is accessed, its timestamp gets updated in the tracking array.
* When it's time to evict some pages, the algorithm randomly chooses 5 indexes from the tracking array and evicts a page with the latest timestamp. If some of the indexes point to non-data pages (index or system pages) then the algorithm picks another.

To get more details about the algorithm implementation refer to `DataPageEvictionMode` javadoc.
[block:api-header]
{
  "title": "On-heap Cache Eviction Policies"
}
[/block]
Eviction policies control the maximum number of elements that can be stored in a cache on-heap memory.  Whenever maximum on-heap cache size is reached, entries are evicted into [off-heap space](doc:off-heap-memory), if one is enabled. 

Some eviction policies support batch eviction and eviction by memory size limit. If batch eviction is enabled than eviction starts when cache size becomes `batchSize` elements greater than the maximum cache size. In this cases `batchSize` entries will be evicted. If eviction by memory size limit is enabled then eviction starts when size of cache entries in bytes becomes greater than the maximum memory size.
[block:callout]
{
  "type": "info",
  "body": "Batch eviction is supported only if maximum memory limit isn't set."
}
[/block]
In Ignite eviction policies are pluggable and are controlled via `EvictionPolicy` interface. An implementation of eviction policy is notified of every cache change and defines the algorithm of choosing the entries to evict from cache. 
[block:callout]
{
  "type": "info",
  "body": "If your data set can fit in memory, then eviction policy will not provide any benefit and should be disabled, which is the default behavior."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Least Recently Used (LRU)"
}
[/block]
LRU eviction policy is based on [Least Recently Used (LRU)](http://en.wikipedia.org/wiki/Cache_algorithms#Least_Recently_Used) algorithm, which ensures that the least recently used entry (i.e. the entry that has not been touched the longest) gets evicted first. 

Supports batch eviction and eviction by memory size limit.
[block:callout]
{
  "type": "success",
  "body": "LRU eviction policy nicely fits most of the use cases for caching. Use it whenever in doubt."
}
[/block]
This eviction policy is implemented by `LruEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n    ...\n    <property name=\"evictionPolicy\">\n        <!-- LRU eviction policy. -->\n        <bean class=\"org.apache.ignite.cache.eviction.lru.LruEvictionPolicy\">\n            <!-- Set the maximum cache size to 1 million (default is 100,000). -->\n            <property name=\"maxSize\" value=\"1000000\"/>\n        </bean>\n    </property>\n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new LruEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "First In First Out (FIFO)"
}
[/block]
FIFO eviction policy is based on [First-In-First-Out (FIFO)](https://en.wikipedia.org/wiki/FIFO) algorithm which ensures that entry that has been in cache the longest will be evicted first. It is different from `LruEvictionPolicy` because it ignores the access order of entries. 

Supports batch eviction and eviction by memory size limit.

This eviction policy is implemented by `FifoEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n    ...\n    <property name=\"evictionPolicy\">\n        <!-- FIFO eviction policy. -->\n        <bean class=\"org.apache.ignite.cache.eviction.fifo.FifoEvictionPolicy\">\n            <!-- Set the maximum cache size to 1 million (default is 100,000). -->\n            <property name=\"maxSize\" value=\"1000000\"/>\n        </bean>\n    </property>\n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new FifoEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Sorted"
}
[/block]
Sorted eviction policy is similar to FIFO eviciton policy with the difference that entries order is defined by default or user defined comparator and ensures that the minimal entry (i.e. the entry that has integer key with smallest value) gets evicted first.

Default comparator uses cache entries keys for comparison that imposes a requirement for keys to implement `Comparable` interface. User can provide own comparator implementation which can use keys, values or both for entries comparison.

Supports batch eviction and eviction by memory size limit.

This eviction policy is implemented by `SortedEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n    ...\n    <property name=\"evictionPolicy\">\n        <!-- Sorted eviction policy. -->\n        <bean class=\"org.apache.ignite.cache.eviction.sorted.SortedEvictionPolicy\">\n            <!-- Set the maximum cache size to 1 million (default is 100,000) and use default comparator. -->\n            <property name=\"maxSize\" value=\"1000000\"/>\n        </bean>\n    </property>\n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new SortedEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Random"
}
[/block]
Random eviction policy which randomly chooses entries to evict. This eviction policy is mainly used for debugging and benchmarking purposes.

This eviction policy is implemented by `RandomEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n    ...\n    <property name=\"evictionPolicy\">\n        <!-- Random eviction policy. -->\n        <bean class=\"org.apache.ignite.cache.eviction.random.RandomEvictionPolicy\">            <!-- Set the maximum cache size to 1 million (default is 100,000). -->\n            <property name=\"maxSize\" value=\"1000000\"/>\n        </bean>\n    </property>\n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new RandomEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]