* [Overview](#overview)
* Page-Based Evicton](#page-based-eviction)
 * [Random-LRU Mode](#section-random-lru-mode)
 * [Random-2-LRU Mode](#section-random-2-lru-mode)
* [On-heap Cache Eviction Policies](#on-heap-cache-eviction-policies)
 * [Least Recently Used](#section-least-recently-used-lru-)
 * [First In First Out](#section-first-in-first-out-fifo-)
 * [Sorted](#section-sorted)
 * [Random](#section-random)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite supports two distinct data eviction policies - **page-based eviction** for the [off-heap page memory](doc:page-memory) and **cache entries based eviction** for the optional [page memory's on-heap cache](doc:page-memory#section-on-heap-caching). 

Page-based eviction is configured via page memory policies, as explained in the next section below.  Cache entries based eviction can be enabled by using the eviction policies explained in the [on-heap cache eviction policies](#on-heap-cache-eviction-policies) section of this document.
[block:api-header]
{
  "title": "Page-Based Eviction"
}
[/block]
[Page Memory](doc:page-memory) consists of one or more memory pools configured by `MemoryPolicyConfigurations`. By default, a pool constantly grows in size until its maximum size is reached.

To avoid possible pool exhaustion, you might need to set one of the data page eviction modes via  the `MemoryPolicyConfiguration.setPageEvictionMode(...)` configuration parameter. The eviction modes track data pages usage and evict some of them according to a mode's implementation.

## Random-LRU Mode

To enable Random-LRU eviction algorithm, pass `DataPageEvictionMode.RANDOM_LRU` value to a respective `MemoryPolicyConfiguration` as shown in the example below: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.MemoryConfiguration\">\n  <!-- Defining additional memory poolicies. -->\n  <property name=\"memoryPolicies\">\n    <list>\n      <!--\n          Defining a policy for 20 GB memory pool with RANDOM_LRU eviction.\n      -->\n      <bean class=\"org.apache.ignite.configuration.MemoryPolicyConfiguration\">\n        <property name=\"name\" value=\"20GB_Pool_Eviction\"/>\n        <!-- Total size of 20 GB. -->\n        <property name=\"size\" value=\"#{20L * 1024 * 1024 * 1024}\"/>\n        <!-- Enabling RANDOM_LRU eviction. -->\n        <property name=\"pageEvictionMode\" value=\"RANDOM_LRU\"/>\n      </bean>\n    </list>\n    ...\n  </property>\n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "// Defining additional memory poolicies.\nMemoryConfiguration memCfg = new MemoryConfiguration();\n\n// Defining a policy for 20 GB memory pool with RANDOM_LRU eviction.\nMemoryPolicyConfiguration memPlc = new MemoryPolicyConfiguration();\n\nmemPlc.setName(\"20GB_Pool_Eviction\");\n\n// Total size of 20 GB.\nmemPlc.setSize(20L * 1024 * 1024 * 1024);\n\n// Enabling RANDOM_LRU eviction.\nmemPlc.setPageEvictionMode(DataPageEvictionMode.RANDOM_LRU);\n        \n// Setting the new memory policy.\nmemCfg.setMemoryPolicies(memPlc);",
      "language": "java"
    }
  ]
}
[/block]
Random-LRU algorithm works the following way:
* Once a memory pool defined by a memory policy is configured, an off-heap array is allocated to track 'last usage' timestamp for every individual data page.
* When a data page is accessed, its timestamp gets updated in the tracking array.
* When it's time to evict some pages, the algorithm randomly chooses 5 indexes from the tracking array and evicts a page with the latest timestamp. If some of the indexes point to non-data pages (index or system pages) then the algorithm picks another.

## Random-2-LRU Mode

To enable Random-2-LRU eviction algorithm, which is a scan-resistant version of Random-LRU, pass `DataPageEvictionMode.RANDOM_2_LRU` value to a respective `MemoryPolicyConfiguration` as it's shown in the example below: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.MemoryConfiguration\">\n  <!-- Defining additional memory poolicies. -->\n  <property name=\"memoryPolicies\">\n    <list>\n      <!--\n          Defining a policy for 20 GB memory pool with RANDOM_2_LRU eviction.\n      -->\n      <bean class=\"org.apache.ignite.configuration.MemoryPolicyConfiguration\">\n        <property name=\"name\" value=\"20GB_Pool_Eviction\"/>\n        <!-- Total size of 20 GB. -->\n        <property name=\"size\" value=\"#{20L * 1024 * 1024 * 1024}\"/>\n        <!-- Enabling RANDOM_2_LRU eviction. -->\n        <property name=\"pageEvictionMode\" value=\"RANDOM_2_LRU\"/>\n      </bean>\n    </list>\n    ...\n  </property>\n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "// Defining additional memory poolicies.\nMemoryConfiguration memCfg = new MemoryConfiguration();\n\n// Defining a policy for 20 GB memory pool with RANDOM_LRU eviction.\nMemoryPolicyConfiguration memPlc = new MemoryPolicyConfiguration();\n\nmemPlc.setName(\"20GB_Pool_Eviction\");\n\n// Total size of 20 GB.\nmemPlc.setSize(20L * 1024 * 1024 * 1024);\n\n// Enabling RANDOM_2_LRU eviction.\nmemPlc.setPageEvictionMode(DataPageEvictionMode.RANDOM_2_LRU);\n        \n// Setting the new memory policy.\nmemCfg.setMemoryPolicies(memPlc);",
      "language": "java"
    }
  ]
}
[/block]
Random-2-LRU differs from Random-LRU only in a way that two latest access timestamps are stored for every data page. At the eviction time, a minimum between two latest timestamps is taken for further comparison with minimums of other pages that chosen as eviction candidates. Random-LRU-2 outperforms LRU by resolving "one-hit wonder" problem - if a data page is accessed rarely, but accidentally accessed once, it's protected from eviction for a long time.
[block:callout]
{
  "type": "success",
  "title": "Eviction Triggering",
  "body": "By default, a data pages eviction algorithm is triggered when the total memory pool consumption gets to 90%. Use `MemoryPolicyConfiguration.setEvictionThreshold(...)` parameter if you need to initiate the eviction earlier or later."
}
[/block]

[block:api-header]
{
  "title": "On-Heap Cache Entries Based Eviction"
}
[/block]
[Page Memory](doc:page-memory) allows storing hot cache entries in Java heap if [on-heap caching](https://apacheignite.readme.io/docs/page-memory#section-on-heap-caching) feature is enabled via `CacheConfiguration.setOnheapCacheEnabled(...)`. Once the on-heap cache is turned on you might need to manage its grows and to do that use one of cache entries eviction policies described below.

Eviction policies control the maximum number of elements that can be stored in a cache on-heap memory.  Whenever maximum on-heap cache size is reached, entries are evicted from Java heap. 

Some eviction policies support batch eviction and eviction by memory size limit. If batch eviction is enabled than eviction starts when cache size becomes `batchSize` elements greater than the maximum cache size. In this cases `batchSize` entries will be evicted. If eviction by memory size limit is enabled then eviction starts when size of cache entries in bytes becomes greater than the maximum memory size.
[block:callout]
{
  "type": "info",
  "body": "Batch eviction is supported only if maximum memory limit isn't set."
}
[/block]
In Apache Ignite eviction policies are pluggable and are controlled via `EvictionPolicy` interface. An implementation of eviction policy is notified of every cache change and defines the algorithm of choosing the entries to evict from on-heap cache of the page memory. 

## Least Recently Used (LRU)

LRU eviction policy is based on [Least Recently Used (LRU)](http://en.wikipedia.org/wiki/Cache_algorithms#Least_Recently_Used) algorithm, which ensures that the least recently used entry (i.e. the entry that has not been touched the longest) gets evicted first. 

Supports batch eviction and eviction by memory size limit.
[block:callout]
{
  "type": "success",
  "body": "LRU eviction policy nicely fits most of the use cases for on-heap caching. Use it whenever in doubt."
}
[/block]
This eviction policy is implemented by `LruEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n  \n  <!-- Enabling on-heap caching for this distributed cache. -->\n  <property name=\"onheapCacheEnabled\" value=\"true\"/>\n  \n  <property name=\"evictionPolicy\">\n  \t<!-- LRU eviction policy. -->\n    <bean class=\"org.apache.ignite.cache.eviction.lru.LruEvictionPolicy\">\n    \t<!-- Set the maximum cache size to 1 million (default is 100,000). -->\n      <property name=\"maxSize\" value=\"1000000\"/>\n    </bean>\n  </property>\n  \n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Enabling on-heap caching for this distributed cache.\ncacheCfg.setOnheapCacheEnabled(true);\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new LruEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
## First In First Out (FIFO)

FIFO eviction policy is based on [First-In-First-Out (FIFO)](https://en.wikipedia.org/wiki/FIFO) algorithm which ensures that entry that has been in the on-heap cache the longest will be evicted first. It is different from `LruEvictionPolicy` because it ignores the access order of entries. 

Supports batch eviction and eviction by memory size limit.

This eviction policy is implemented by `FifoEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n  \n  <!-- Enabling on-heap caching for this distributed cache. -->\n  <property name=\"onheapCacheEnabled\" value=\"true\"/>\n  \n  <property name=\"evictionPolicy\">\n  \t<!-- FIFO eviction policy. -->\n    <bean class=\"org.apache.ignite.cache.eviction.fifo.FifoEvictionPolicy\">\n    \t<!-- Set the maximum cache size to 1 million (default is 100,000). -->\n      <property name=\"maxSize\" value=\"1000000\"/>\n    </bean>\n  </property>\n  \n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Enabling on-heap caching for this distributed cache.\ncacheCfg.setOnheapCacheEnabled(true);\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new FifoEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
## Sorted

Sorted eviction policy is similar to FIFO eviciton policy with the difference that entries order is defined by default or user defined comparator and ensures that the minimal entry (i.e. the entry that has integer key with smallest value) gets evicted first.

Default comparator uses cache entries keys for comparison that imposes a requirement for keys to implement `Comparable` interface. User can provide own comparator implementation which can use keys, values or both for entries comparison.

Supports batch eviction and eviction by memory size limit.

This eviction policy is implemented by `SortedEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n  \n  <!-- Enabling on-heap caching for this distributed cache. -->\n  <property name=\"onheapCacheEnabled\" value=\"true\"/>\n  \n  <property name=\"evictionPolicy\">\n  \t<!-- Sorted eviction policy. -->\n    <bean class=\"org.apache.ignite.cache.eviction.sorted.SortedEvictionPolicy\">\n    \t<!--\n \t\t\t\t\tSet the maximum cache size to 1 million (default is 100,000)\n\t\t\t\t\tand use default comparator.\n\t\t\t-->\n      <property name=\"maxSize\" value=\"1000000\"/>\n    </bean>\n  </property>\n  \n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Enabling on-heap caching for this distributed cache.\ncacheCfg.setOnheapCacheEnabled(true);\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new SortedEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
## Random

Random eviction policy which randomly chooses entries to evict. This eviction policy is mainly used for debugging and benchmarking purposes.

This eviction policy is implemented by `RandomEvictionPolicy` and can be configured via `CacheConfiguration`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.cache.CacheConfiguration\">\n  <property name=\"name\" value=\"myCache\"/>\n  \n  <!-- Enabling on-heap caching for this distributed cache. -->\n  <property name=\"onheapCacheEnabled\" value=\"true\"/>\n  \n  <property name=\"evictionPolicy\">\n  \t<!-- Random eviction policy. -->\n    <bean class=\"org.apache.ignite.cache.eviction.random.RandomEvictionPolicy\">       \t<!-- Set the maximum cache size to 1 million (default is 100,000). -->\n      <property name=\"maxSize\" value=\"1000000\"/>\n    </bean>\n  </property>\n  \n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\n// Enabling on-heap caching for this distributed cache.\ncacheCfg.setOnheapCacheEnabled(true);\n\n// Set the maximum cache size to 1 million (default is 100,000).\ncacheCfg.setEvictionPolicy(new RandomEvictionPolicy(1000000));\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]