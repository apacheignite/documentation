* [Overview](doc:page-memory#overview)
* [Page Memory](doc:page-memory#page-memory)
* [Memory Policies](doc:page-memory#memory-policies)
* [Eviction Modes](doc:page-memory#eviction-modes)
* [On-heap Caching](doc:page-memory#on-heap-caching)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Starting with version 2.0, Apache Ignite has introduced the new off-heap memory architecture with possible on-heap caching as it's shown on the picture below: 
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/353f4c0-aM4uHn8wRMmmtFvkZndL_off-heap-memory.png",
        "aM4uHn8wRMmmtFvkZndL_off-heap-memory.png",
        450,
        354,
        "#0a0907"
      ]
    }
  ]
}
[/block]
The new memory architecture has the following benefits:

* Predictable memory consumption. There is a way to say precisely how much memory you are ready to give to an Apache Ignite node process and arrange data sets of specific Ignite caches across memory regions of different characteristics such as total capacity.
* Automatic memory defragmentation. Apache Ignite uses all the available memory as efficient as possible and executes defragmentation routines in background not affecting the performance of your application. 
* No stop-the-world (STW) garbage collection (GC) pauses caused by Apache Ignite platform architecture. Application's code is the only one possible source of STW GC pauses.
* Profoundly improves the performance of and memory utilization by Apache Ignite SQL engine.
* Transparent swapping. The page memory can be mapped to a memory-mapped file offloading swapping related activity to an underlying operating system.
* An ability to integrate with a file system in order to persist memory pages on disk for durability purpose and create cluster-wide snapshots (backups).
[block:api-header]
{
  "type": "basic",
  "title": "Page Memory"
}
[/block]
Page memory is a manageable off-heap based memory architecture that splits all continuously allocated memory slabs into pages of fixed size.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/0aa5c62-image1.JPG",
        "image1.JPG",
        3264,
        2448,
        "#cdc8c4"
      ]
    }
  ]
}
[/block]
Both key-value entries and SQL indexes are stored in memory pages of different types, that are called data pages and index pages respectively, and the pages as the whole page memory are managed by Apache Ignite transparently to the end users. 

Usually, a single data page stores multiple key-value entries in order to use the memory as efficient as possible and to avoid the memory fragmentation. At the time when a new key-value entry is being added to a cache, the page memory will look up a page that can fit the whole entry and puts it there. If an entry's total size exceeds the page size configured via `MemoryConfiguration.setPageSize(..)` parameter then the entry will occupy more than one data page.
[block:callout]
{
  "type": "info",
  "title": "Entry Ownership by Data Page",
  "body": "A key-value entry might not be bound to a specific page all the times. For instance, if during an update the entry expands and its current page can no longer fit it then the page memory will search for a new page that has enough room to take the updated entry and will move the entry there."
}
[/block]
The page memory can encompass multiple continuous memory slabs with distinct properties such as slab size or an eviction policy (refer to [memory policies](doc:page-memory#memory-policies) section below). However, by default, an Apache Ignite node sets up and initiates a single continuous memory slab that will be used by all the Apache Ignite caches defined in your configuration.

## Configuration Parameters

To alter global page memory settings such as page size use `org.apache.ignite.configuration.MemoryConfiguration` that is passed via `IgniteConfiguration.setMemoryConfiguration(...)` method. Below you can see all the available parameters:
[block:parameters]
{
  "data": {
    "h-0": "Parameter Name",
    "h-1": "Description",
    "h-2": "Default Value",
    "0-0": "`setPageSize(...)`",
    "0-1": "Sets default page size.",
    "0-2": "2 KB",
    "1-0": "`setDefaultMemoryPolicyName(...)`",
    "1-1": "Sets default memory policy's name. By default every cache is bound to a memory slab instantiated with this policy.\n\nRefer to [memory policies](doc:page-memory#memory-policies) section to learn more about memory policies.",
    "1-2": "`null`",
    "2-0": "`setMemoryPolicies(...)`",
    "2-1": "Sets a list of all memory policies available in the cluster.\n\nRefer to [memory policies](doc:page-memory#memory-policies) section to learn more about memory policies.",
    "2-2": "An empty array. A configuration that is used to create the default region is not stored there.",
    "3-0": "`setSystemCacheMemorySize(...)`",
    "3-1": "Sets size for Apache Ignite's internal system cache.",
    "3-2": "100 MB",
    "4-0": "`setConcurrencyLevel(...)`",
    "4-1": "Sets the number of concurrent segments in Apache Ignite internal page mapping tables",
    "4-2": "A total number of available CPUs times 4."
  },
  "cols": 3,
  "rows": 5
}
[/block]
An example below shows how to change page size and concurrency level parameters using `MemoryConfiguration`:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <property name=\"memoryConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.MemoryConfiguration\">\n            <!-- Setting the concurrency level -->\n            <property name=\"concurrencyLevel\" value=\"4\"/>\n                \n            <!-- Setting the page size to 4 KB -->\n            <property name=\"pageSize\" value=\"4096\"/>\n        </bean>\n    </property>\n  \n  <!--- Additional settings ---->\n</bean>",
      "language": "xml",
      "name": ""
    },
    {
      "code": "// Ignite configuration.\nIgniteConfiguration cfg = new IgniteConfiguration();\n\n// Page memory configuration.\nMemoryConfiguration memCfg = new MemoryConfiguration();\n\n// Altering the concurrency level.\nmemCfg.setConcurrencyLevel(4);\n\n// Changing the page size to 4 KB.\nmemCfg.setPageSize(4096);\n        \n// Applying the new page memory configuration.\ncfg.setMemoryConfiguration(memCfg);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Memory Policies"
}
[/block]
By default, the page memory initiates a single continuous memory slab that is used by all the caches configured and deployed in your cluster. However, there is a way to define multiple memory slabs with various parameters and custom behavior relying on memory policies' API.

A memory policy is a set of configuration parameters, that are exposed through `org.apache.ignite.configuration.MemoryPolicyConfiguration`, like slab size, an eviction policy, a swapping file and more.

For instance, to configure a 500 MB memory slab with data pages eviction we need to define the memory policy below:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n   <!-- Page memory configuration -->   \n   <property name=\"memoryConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.MemoryConfiguration\">\n        <!-- Defining a custom memory policy. -->\n        <property name=\"memoryPolicies\">\n          <list>\n            <!-- 500 MB total size and RANDOM_2_LRU eviction algorithm. -->\n            <bean class=\"org.apache.ignite.configuration.MemoryPolicyConfiguration\">\n              <property name=\"name\" value=\"500MB_Region_Eviction\"/>\n              <!-- 500 MB total size. -->\n              <property name=\"size\" value=\"#{500 * 1024 * 1024}\"/>\n              <!-- Enabling data pages eviction. -->\n              <property name=\"pageEvictionMode\" value=\"RANDOM_2_LRU\"/>\n            </bean>\n          </list>\n        </property>\n      </bean>\n   </property>\n  \n  <!-- The rest of the configuration. -->\n  <!-- ....... -->\n</bean>",
      "language": "xml"
    },
    {
      "code": "// Ignite configuration.\nIgniteConfiguration cfg = new IgniteConfiguration();\n\n// Page memory configuration.\nMemoryConfiguration memCfg = new MemoryConfiguration();\n\n// Creating a custom memory policy for a new memory slab.\nMemoryPolicyConfiguration plCfg = new MemoryPolicyConfiguration();\n\n// Policy/region name.\nplCfg.setName(\"500MB_Region_Eviction\");\n\n// Setting slab's size to 500 MB.\nplCfg.setSize(500 * 1024 * 1024);\n\n// Setting data pages eviction algorithm.\nplCfg.setPageEvictionMode(DataPageEvictionMode.RANDOM_2_LRU);\n\n// Applying the memory policy.\nmemCfg.setMemoryPolicies(plCfg);\n        \n// Applying the new page memory configuration.\ncfg.setMemoryConfiguration(memCfg);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
Right after that, an Apache Ignite cache can be mapped to the slab defined by the memory policy above. To achieve this, the name of the policy has to be passed as a parameter of `org.apache.ignite.configuration.CacheConfiguration.setMemoryPolicyName(...)` method: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <!-- Page memory and other configuration parameters. -->\n    <!-- ....... -->\n  \n    <property name=\"cacheConfiguration\">\n       <list>\n           <!-- Cache that is mapped to non-default memory slab -->\n           <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n              <!--\n                   Setting a memory policy name to bind to a specific slab.\n               -->\n               <property name=\"memoryPolicyName\" value=\"500MB_Region_Eviction\"/>\n               <!-- Cache unique name. -->\n               <property name=\"name\" value=\"SampleCache\"/>\n             \n               <!-- Additional cache configuration parameters -->\n           </bean>\n       </list>\n    </property>\n    \n    <!-- The rest of the configuration. -->\n    <!-- ....... -->\n</bean>  ",
      "language": "xml"
    },
    {
      "code": "// Ignite configuration.\nIgniteConfiguration cfg = new IgniteConfiguration();\n\n// Page memory configuration and the rest of the configuration.\n// ....\n\n// Creating a cache configuration.\nCacheConfiguration cacheCfg = new CacheConfiguration();\n\n// Setting a memory policy name to bind to a specific memory region.\ncacheCfg.setMemoryPolicyName(\"500MB_Region_Eviction\");\n        \n// Setting the cache name.\ncacheCfg.setName(\"SampleCache\");\n\n// Applying the cache configuration.\ncfg.setCacheConfiguration(cacheCfg);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
Once Apache Ignite cluster is started with this configuration, the page memory will allocate 500 MB slab in addition to the default one and all the data as well as possible indexes, omitted in this example, of `SampleCache` will reside in that region. The rest of the caches you might have in your deployment will be bound to the default slab unless you map them to another memory slabs explicitly using the technique shown above.
[block:callout]
{
  "type": "success",
  "title": "Changing Default Memory Policy",
  "body": "The default memory slab is instantiated with the parameters of the default memory policy prepared by `org.apache.ignite.configuration.MemoryConfiguration.createDefaultPolicyConfig()` method. If you need to change some parameters of the default slab then follow the steps below:\n* Create a new memory policy with a custom name and parameters.\n* Pass the name of the policy to `org.apache.ignite.configuration.MemoryConfiguration.\nsetDefaultMemoryPolicyName(...)` method."
}
[/block]
Refer to [memory policies example](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/datagrid/MemoryPoliciesExample.java) to see how to configure and use multiple slabs in your cluster.

## Configuration Parameters

`org.apache.ignite.configuration.MemoryPolicyConfiguration` supports the following parameters:
[block:parameters]
{
  "data": {
    "h-0": "Parameter Name",
    "h-1": "Description",
    "h-2": "Default Value",
    "0-0": "`setName(...)`",
    "0-1": "Unique memory policy name.",
    "0-2": "Required parameter.",
    "1-0": "`setSize(...)`",
    "1-1": "Sets total slab size defined by this memory policy.\n\nThe total size can not be less than 1 MB due to internal requirements.\n\nIf the overall memory usage goes beyond this parameter an out of memory exception will be thrown. To avoid this use an eviction algorithm or set this parameter to a bigger value.",
    "1-2": "Required parameter.",
    "2-0": "`setSwapFilePath(...)`",
    "2-1": "A path to the memory-mapped file the slab defined by this memory policy will be mapped to. Having this path set, allows relying on swapping capabilities of an underlying operating system for the slab.",
    "2-2": "Disabled by default.",
    "3-0": "`setPageEvictionMode(...)`",
    "3-1": "Sets one of the data pages eviction algorithms available for usage. Refer to [eviction policies](doc:evictions) for more details.",
    "3-2": "Disabled by default.",
    "4-0": "`setEvictionThreshold(...)`",
    "4-1": "A threshold for memory pages eviction initiation. For instance, if the threshold is 0.9 it means that the page memory will start the eviction only after 90% of the slab (defined by this policy) is occupied.",
    "4-2": "`0.9`",
    "5-0": "`setEmptyPagesPoolSize(...)`",
    "5-1": "Specifies the minimal number of empty pages to be present in reuse lists for this memory policy. This parameter ensures that Apache Ignite will be able to successfully evict old data entries when the size of a (key, value) pair is slightly larger than page size / 2.\n\nIncrease this parameter if a cache can contain very big entries (total size of pages in this pool should be enough to contain largest cache entry).",
    "5-2": "`100`"
  },
  "cols": 3,
  "rows": 6
}
[/block]

[block:api-header]
{
  "title": "Eviction Modes"
}
[/block]
Memory policy allows setting up various eviction modes for data pages that store key-value entries. To learn more about the available algorithms refer to [this](doc:evictions) documentation.
[block:api-header]
{
  "title": "On-heap Caching"
}
[/block]
The page memory is an off-heap memory that allocates all the memory regions outside of Java heap and stores cache entries there. However, there is a way to enable on-heap caching for the cache entries by setting `org.apache.ignite.configuration.CacheConfiguration.setOnheapCacheEnabled(...)` to `true`.

The on-heap caching is useful for the scenarios when you do a lot of cache reads on server nodes side working with cache entries in the [binary form](doc:binary-marshaller) or provoking cache entries deserialization. For instance, this might happen when a distributed computation or deployed service gets some data from caches for further processing according to the implemented logic.
[block:callout]
{
  "type": "success",
  "title": "On-Heap Cache Size",
  "body": "To manage the on-heap cache size and avoid its constant grow make sure to enable one of available [cache entries based eviction policies](doc:evictions)."
}
[/block]