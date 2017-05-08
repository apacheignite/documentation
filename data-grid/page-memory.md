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
Starting with version 2.0, Apache Ignite has introduced a new off-heap memory architecture with possible on-heap caching. 
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

* Predictable memory consumption - There is a way to say precisely how much memory you are ready to give to an Apache Ignite node process and arrange data sets of specific Ignite caches across memory regions of different characteristics such as total capacity.
* Automatic memory defragmentation -  Apache Ignite uses all the available memory as efficiently as possible and executes defragmentation routines in the background, without affecting the performance of your application. 
* No stop-the-world (STW) garbage collection (GC) pauses caused by Apache Ignite platform architecture. Application's code is the only possible source of STW GC pauses.
* Immensely improves the performance and memory utilization by Apache Ignite SQL engine.
* Transparent swapping. The page memory can be mapped to a memory-mapped file offloading swapping related activity to an underlying operating system.
* Ability to integrate with a file system in order to persist memory pages on disk for durability purpose and create cluster-wide snapshots (backups).
[block:api-header]
{
  "type": "basic",
  "title": "Page Memory"
}
[/block]
Page memory is a manageable off-heap based memory architecture that is split into pages of fixed size. Let's take a look at the picture below and try to understand more about this memory architecture.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/0bf1bbf-Page-Memory-Diagram-v3.png",
        "Page-Memory-Diagram-v3.png",
        800,
        787,
        "#cfc5c7"
      ]
    }
  ]
}
[/block]
## Memory Regions

The whole page memory of an individual Apache Ignite node can consist of one or many memory regions. A memory region is a logical expandable area that is configured with a [memory policy](doc:page-memory#memory-policies). The regions can vary in size, eviction policies and other parameters explained in the memory policy section below.   

## Memory Chunk

Every memory region has the maximum size it can grow to. The region expands to its maximum boundary allocating continuous memory chunks.
[block:callout]
{
  "type": "success",
  "title": "Default Maximum Size",
  "body": "If a memory region is not limited explicitly via respective `MemoryPolicyConfiguration`, then it can take up to 80% of the RAM available on your machine."
}
[/block]
A memory chunk is a physical continuous byte array obtained from the operating system. The chunk is divided into pages of fixed size. There are several types of pages that can reside in the chunk:
  
## Data Page

A data page stores cache entries you put into Apache Ignite caches from an application side (data pages are colored in green in the picture above).

Usually, a single data page holds multiple key-value entries in order to use the memory as efficiently as possible and avoid memory fragmentation. When a new key-value entry is being added to a cache, the page memory will look up a page that can fit the whole entry and puts it there. However, if an entry's total size exceeds the page size configured via the`MemoryConfiguration.setPageSize(..)` parameter, then the entry will occupy more than one data page.
[block:callout]
{
  "type": "info",
  "title": "Entry Ownership by Data Page",
  "body": "A key-value entry might not be bound to a specific page all the time. For instance, if during an update the entry expands and its current page can no longer fit it, then the page memory will search for a new data page that has enough room to take the updated entry and will move the entry there."
}
[/block]
## B+ Tree and Index Page

SQL indexes defined and used in an application are arranged and maintained in the form of a B+ tree data structure. For every unique index that is declared in an SQL schema, Apache Ignite instantiates and manages a dedicated B+ tree instance. 
[block:callout]
{
  "type": "success",
  "title": "Hash Index",
  "body": "Cache entries' keys are also referenced from the B+ tree data structure. They're ordered by hash code value."
}
[/block]
As shown in the picture above, the whole purpose of a B+ tree is to link and order the index pages that are allocated and stored in random physical locations of the page memory. Internally, an index page contains all the information needed to locate the index's value, cache entry's offset in a data page an index refers to, and references to other index pages in order to traverse the tree (index pages are colored in purple in the picture above). 

B+ tree Meta Page is needed to get to the root of a specific B+ tree and to its layers for efficient execution of range queries. For instance, when `myCache.get(keyA)` operation is executed, it will trigger the following execution flow on an Apache Ignite node:
* Apache Ignite will look for a memory region to which `myCache` belongs to.
* Inside that memory region, a meta page of a B+ tree that orders keys of `myCache` will be located.
* Hash code of `keyA` will be calculated and an index page the key belongs to will be searched for in the B+ tree.
* If the corresponding index page is not found, then it means the key-value pair doesn't exist in `myCache` and Apache Ignite will return `null` as a result of the `myCache.get(keyA)` operation.
* If the index page exists, then it will contain all the information needed to find the data page of the cache entry `keyA` refers to.
* The cache entry is taken from the data page and returned to your application.
 
## Free Lists Metadata and Structure

The execution flow from the previous section explains how a cache entry is looked up in the page memory when you want to get it from your application. However, how does the page memory know where to put a new cache entry if an operation like `myCache.put(keyA, valueA)` is called?

In this scenario, the page memory relies on free lists data structures. Basically, a free list is a doubly linked list that stores references to memory pages of approximately equal free space. For instance, there is a free list that stores all the data pages that have up to 75% free space and a list that keeps track of the index pages with 25% capacity left. Data and index pages are tracked in separate free lists.

Keeping this in mind, the execution flow of `myCache.put(keyA, valueA)` operation on an Apache Ignite node, that is a primary or backup node for the entry, will be more or less the following:
* Apache Ignite will look for a memory region to which `myCache` belongs to.
* Inside that memory region, a meta page of a B+ tree that orders keys of `myCache` will be located.
* Hash code of `keyA` will be calculated and an index page the key belongs to will be searched for in the B+ tree.
* If the index page is not found, then it will be requested from one of the free lists depending on the index size. A targeted free list will be found by the help of a free list meta page. Once the index page is provided, it will be added to the B+ tree hierarchy.
* If the index page doesn't already contain a data page the cache entry has to be placed into, then the data page will be given by one of the free lists depending on the total cache entry size. A reference to the data page will be added from the index page.
* The cache entry is added into the data page.

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
    "0-2": "`2` KB",
    "1-0": "`setDefaultMemoryPolicyName(...)`",
    "1-1": "Sets default memory policy's name. By default every cache is bound to a memory region instantiated with this policy.\n\nRefer to [memory policies](doc:page-memory#memory-policies) section to learn more about memory policies.",
    "1-2": "`default`",
    "3-0": "`setMemoryPolicies(...)`",
    "3-1": "Sets a list of all memory policies available in the cluster.\n\nRefer to [memory policies](doc:page-memory#memory-policies) section to learn more about memory policies.",
    "3-2": "An empty array. A configuration that is used to create the default region is not stored there.",
    "4-0": "`setSystemCacheInitialSize(...)`",
    "4-1": "Sets initial size of a memory region reserved for system cache.",
    "4-2": "`40` MB",
    "6-0": "`setConcurrencyLevel(...)`",
    "6-1": "Sets the number of concurrent segments in Apache Ignite internal page mapping tables",
    "6-2": "A total number of available CPUs times 4.",
    "2-0": "`setDefaultMemoryPolicySize(...)`",
    "2-1": "Sets the size of the default memory region which is created automatically.\n\nIf the proper is not set then the default region can consume up to 80% of RAM available on a local machine.",
    "2-2": "`80%` of RAM",
    "5-0": "`setSystemCacheMaxSize(...)`",
    "5-1": "Sets maximum size of a memory region reserved for system cache.\n\nThe total size should not be less than 10 MB due to internal data structures overhead.",
    "5-2": "`100` MB"
  },
  "cols": 3,
  "rows": 7
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
By default, the page memory initiates a single expandable memory region that can take up to 80% of all the memory available on a local machine. However, there is a way to define multiple memory regions with various parameters and custom behavior relying on memory policies' API.

A memory policy is a set of configuration parameters, that are exposed through `org.apache.ignite.configuration.MemoryPolicyConfiguration`, like initial and maximum region size, an eviction policy, a swapping file and more.

For instance, to configure a 500 MB memory region with enabled data pages eviction we need to define a memory policy like below:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n   <!-- Page memory configuration -->   \n   <property name=\"memoryConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.MemoryConfiguration\">\n        <!-- Defining a custom memory policy. -->\n        <property name=\"memoryPolicies\">\n          <list>\n            <!-- 500 MB total size and RANDOM_2_LRU eviction algorithm. -->\n            <bean class=\"org.apache.ignite.configuration.MemoryPolicyConfiguration\">\n              <property name=\"name\" value=\"500MB_Region_Eviction\"/>\n              <!-- 100 MB initial size. -->\n              <property name=\"initialSize\" value=\"#{100 * 1024 * 1024}\"/>\n              <!-- 500 MB maximum size. -->\n              <property name=\"maxSize\" value=\"#{500 * 1024 * 1024}\"/>\n              <!-- Enabling data pages eviction. -->\n              <property name=\"pageEvictionMode\" value=\"RANDOM_2_LRU\"/>\n            </bean>\n          </list>\n        </property>\n      </bean>\n   </property>\n  \n  <!-- The rest of the configuration. -->\n  <!-- ....... -->\n</bean>",
      "language": "xml"
    },
    {
      "code": "// Ignite configuration.\nIgniteConfiguration cfg = new IgniteConfiguration();\n\n// Page memory configuration.\nMemoryConfiguration memCfg = new MemoryConfiguration();\n\n// Creating a custom memory policy for a new memory region.\nMemoryPolicyConfiguration plCfg = new MemoryPolicyConfiguration();\n\n// Policy/region name.\nplCfg.setName(\"500MB_Region_Eviction\");\n\n// Setting initial size.\nplCfg.setInitialSize(100L * 1024 * 1024);\n\n// Setting maximum size.\nplCfg.setMaxSize(500L * 1024 * 1024);\n\n// Setting data pages eviction algorithm.\nplCfg.setPageEvictionMode(DataPageEvictionMode.RANDOM_2_LRU);\n\n// Applying the memory policy.\nmemCfg.setMemoryPolicies(plCfg);\n        \n// Applying the new page memory configuration.\ncfg.setMemoryConfiguration(memCfg);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
Right after that, an Apache Ignite cache can be mapped to that region. To achieve this, the name of the policy has to be passed as a parameter to `org.apache.ignite.configuration.CacheConfiguration.setMemoryPolicyName(...)` method: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <!-- Page memory and other configuration parameters. -->\n    <!-- ....... -->\n  \n    <property name=\"cacheConfiguration\">\n       <list>\n           <!-- Cache that is mapped to non-default memory region. -->\n           <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n              <!--\n                   Setting a memory policy name to bind to a specific region.\n               -->\n               <property name=\"memoryPolicyName\" value=\"500MB_Region_Eviction\"/>\n               <!-- Cache unique name. -->\n               <property name=\"name\" value=\"SampleCache\"/>\n             \n               <!-- Additional cache configuration parameters -->\n           </bean>\n       </list>\n    </property>\n    \n    <!-- The rest of the configuration. -->\n    <!-- ....... -->\n</bean>  ",
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
Once Apache Ignite cluster is started with this configuration, the page memory will allocate 100 MB region that can grow up to 500 MB. That new region will coexist with the default memory region and all the data as well as indexes, omitted in this example, of `SampleCache` will reside in that region. The rest of the caches you might have in your deployment will be bound to the default memory region unless you map them to another region explicitly using the technique shown above.
[block:callout]
{
  "type": "success",
  "title": "Changing Default Memory Policy",
  "body": "The default memory region is instantiated with the parameters of the default memory policy prepared by `org.apache.ignite.configuration.MemoryConfiguration.createDefaultPolicyConfig()` method. If you need to change some of the parameters then follow the steps below:\n* Create a new memory policy with a custom name and parameters.\n* Pass the name of the policy to `org.apache.ignite.configuration.MemoryConfiguration.\nsetDefaultMemoryPolicyName(...)` method."
}
[/block]
Refer to [memory policies example](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/datagrid/MemoryPoliciesExample.java) to see how to configure and use multiple memory regions in your cluster.

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
    "1-0": "`setInitialSize(...)`",
    "1-1": "Sets initial size of the memory region defined by this memory policy. When the used memory size exceeds this value, new chunks of memory will be allocated until maximum size is reached.",
    "1-2": "`256` MB",
    "3-0": "`setSwapFilePath(...)`",
    "3-1": "A path to the memory-mapped file the memory region defined by this memory policy will be mapped to. Having this path set, allows relying on swapping capabilities of an underlying operating system for the region.",
    "3-2": "Disabled by default.",
    "4-0": "`setPageEvictionMode(...)`",
    "4-1": "Sets one of the data pages eviction algorithms available for usage. Refer to [eviction policies](doc:evictions) for more details.",
    "4-2": "Disabled by default.",
    "5-0": "`setEvictionThreshold(...)`",
    "5-1": "A threshold for memory pages eviction initiation. For instance, if the threshold is 0.9 it means that the page memory will start the eviction only after 90% of the memory region (defined by this policy) is occupied.",
    "5-2": "`0.9`",
    "6-0": "`setEmptyPagesPoolSize(...)`",
    "6-1": "Specifies the minimal number of empty pages to be present in reuse lists for this memory policy. This parameter ensures that Apache Ignite will be able to successfully evict old data entries when the size of a (key, value) pair is slightly larger than page size / 2.\n\nIncrease this parameter if a cache can contain very big entries (total size of pages in this pool should be enough to contain largest cache entry).",
    "6-2": "`100`",
    "2-0": "`setMaxSize(...)`",
    "2-1": "Sets maximum size of the memory region defined by this memory policy.\n\nThe total size should not be less than 10 MB due to the internal data structures overhead.\n\nIf the overall memory usage goes beyond this parameter an out of memory exception will be thrown. To avoid this use an eviction algorithm or set this parameter to a bigger value.",
    "2-2": "`80%` of RAM",
    "7-0": "`setMetricsEnabled(...)`",
    "7-1": "Enables memory metrics aggregation for this region. Refer to [Memory & Cache Metrics](doc:memory-and-cache-metrics)  for more details.",
    "7-2": "`false`"
  },
  "cols": 3,
  "rows": 8
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