* [Overview](doc:page-memory#overview)
* [Page Memory](doc:page-memory#page-memory)
* [Memory Policies](doc:page-memory#memory-policies)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Prior to Apache Ignite 2.0, the data grid's distributed key-value store was built on top and relied on three memory tiers - on-heap tier, off-heap tier, and swap tier. All these memory modes had some tier-specific disadvantages:

* On-heap tier: data stored in this memory layer resided in the Java heap managed by Java VM. If the overall data set's size grew beyond 20 GB on a single Apache Ignite node then you could easily face long stop-the-world garbage collection pauses that in return could cause a significant performance hit or complete ejection of the node from the cluster. Considering that in most modern Apache Ignite use cases the overall memory consumption on individual nodes goes far beyond than 20 GB the off-heap tier was started to be used as a default memory mode to mitigate this on-heap tier issue.

* Off-heap tier: data was stored in a manageable memory region outside of Java heap (off-heap). All the memory allocations and releases were managed by Apache Ignite internally in a fully transparent fashion for the end user. This memory mode helped to resolve all the on-heap tier's issues allowing to store hundreds of gigabytes and petabytes of data on an individual node effortlessly. However, the off-heap tier still had some disadvantages on its own such as uncontrollable memory fragmentation and necessity to maintain SQL rows/indexes cache in Java heap that barred from doing some of the most valuable SQL Grid optimization tasks.

* Swap tier: when there was no more room left in one of the two tiers above, the data could have been swapped to disk to the Apache Ignite swap tier. Frequent interactions with the tier could cause significant performance degradation of the overall application especially if SQL queries were used because considerable amount of data might have been moved between the swap and other memory layers.          

All the memory tiers listed above have been discontinued in Apache Ignite 2.0 in favor of a completely new off-heap based `page memory` that has the following benefits:
* No stop-the-world (STW) garbage collection (GC) pauses caused by Apache Ignite platform architecture. Application's code is the only one possible source of STW GC pauses.
* No memory fragmentation.
* Profoundly improves the performance of and memory utilization by SQL Grid and opens up a room for an enormous number of further optimizations.
* Transparent swapping: the page memory can be mapped to a memory-mapped file offloading swapping related activity to an underlying operating system.
* An ability to integrate with a file system in order to persist memory pages on disk for durability purpose and create cluster-wide snapshots (backups).
[block:api-header]
{
  "type": "basic",
  "title": "Page Memory"
}
[/block]
Page memory is a manageable off-heap based memory architecture that splits all continuously allocated memory regions into pages of fixed size.

Both key-value entries and SQL indexes are stored in memory pages of different types, that are called data pages and index pages respectively, and the pages as the whole page memory are managed by Apache Ignite transparently to the end users. 

Usually, a single data page stores multiple key-value entries in order to use the memory as efficient as possible and to avoid the memory fragmentation. At the time when a new key-value entry is being added to a cache, the page memory will look up a page that can fit the whole entry and puts it there. If an entry's total size exceeds the page size configured via `MemoryConfiguration.setPageSize(..)` parameter then the entry will occupy more than one data page.
[block:callout]
{
  "type": "info",
  "title": "Entry Ownership by Data Page",
  "body": "A key-value entry might not be bound to a specific page all the times. For instance, if during an update the entry expands and its current page can no longer fit it then the page memory will search for a new page that has enough room to take the updated entry and will move the entry there."
}
[/block]
The page memory can encompass multiple continuous memory regions with distinct properties such as region size or an eviction policy (refer to [memory policies](doc:page-memory#memory-policies) section below). However, by default, an Apache Ignite node sets up and initiates a single continuous memory region that will be used by all the Apache Ignite caches defined in your configuration.

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
    "1-1": "Sets default memory policy's name. By default every cache is bound to a memory region instantiated with this policy.\n\nRefer to [memory policies](doc:page-memory#memory-policies) section to learn more about memory policies.",
    "1-2": "`null`",
    "2-0": "`setMemoryPolicies(...)`",
    "2-1": "Sets a list of all memory policies available in the cluster.\n\nRefer to [memory policies](doc:page-memory#memory-policies) section to learn more about memory policies.",
    "2-2": "An empty array. A configuration that is used to create the default region is not stored there.",
    "3-0": "`setSystemCacheMemorySize(...)`",
    "3-1": "Sets size for Apache Ignite's internal system cache.\n\nTODO: what happens if the system cache exceeds a defined value.",
    "3-2": "100 MB",
    "4-0": "`setConcurrencyLevel(...)`",
    "4-1": "TODO",
    "4-2": "TODO"
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
By default, the page memory initiates a single continuous memory region that is used by all the caches configured and deployed in your cluster. However, there is a way to define multiple memory regions with various parameters and custom behavior relying on memory policies' API.

A memory policy is a set of configuration parameters, that are exposed through `org.apache.ignite.configuration.MemoryPolicyConfiguration`, like region size, an eviction policy, a swapping file and more.

For instance, to configure a 500 MB memory region with data pages eviction enabled the following needs to be done:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n   <!-- Page memory configuration -->   \n   <property name=\"memoryConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.MemoryConfiguration\">\n        <!-- Defining a custom memory policy. -->\n        <property name=\"memoryPolicies\">\n          <list>\n            <!-- 500 MB total size and RANDOM_2_LRU eviction algorithm. -->\n            <bean class=\"org.apache.ignite.configuration.MemoryPolicyConfiguration\">\n              <property name=\"name\" value=\"500MB_Region_Eviction\"/>\n              <!-- 500 MB total size. -->\n              <property name=\"size\" value=\"#{500 * 1024 * 1024}\"/>\n              <!-- Enabling data pages eviction. -->\n              <property name=\"pageEvictionMode\" value=\"RANDOM_2_LRU\"/>\n            </bean>\n          </list>\n        </property>\n      </bean>\n   </property>\n  \n  <!-- The rest of the configuration. -->\n  <!-- ....... -->\n</bean>",
      "language": "xml"
    },
    {
      "code": "// Ignite configuration.\nIgniteConfiguration cfg = new IgniteConfiguration();\n\n// Page memory configuration.\nMemoryConfiguration memCfg = new MemoryConfiguration();\n\n// Creating a custom memory policy for a new memory region.\nMemoryPolicyConfiguration plCfg = new MemoryPolicyConfiguration();\n\n// Policy/region name.\nplCfg.setName(\"500MB_Region_Eviction\");\n\n// Setting region's size to 500 MB.\nplCfg.setSize(500 * 1024 * 1024);\n\n// Setting data pages eviction algorithm.\nplCfg.setPageEvictionMode(DataPageEvictionMode.RANDOM_2_LRU);\n\n// Applying the memory policy.\nmemCfg.setMemoryPolicies(plCfg);\n        \n// Applying the new page memory configuration.\ncfg.setMemoryConfiguration(memCfg);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
Right after that, an Apache Ignite cache can be mapped to the memory region defined by the memory policy above. To achieve this, the name of the policy has to be passed as a parameter of `org.apache.ignite.configuration.CacheConfiguration.setMemoryPolicyName(...)` method: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <!-- Page memory and other configuration parameters. -->\n    <!-- ....... -->\n  \n    <property name=\"cacheConfiguration\">\n       <list>\n           <!-- Cache that is mapped to non-default memory region -->\n           <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n              <!--\n                   Setting a memory policy name to bind to a specific\n                   memory region.\n               -->\n               <property name=\"memoryPolicyName\" value=\"500MB_Region_Eviction\"/>\n               <!-- Cache unique name. -->\n               <property name=\"name\" value=\"SampleCache\"/>\n             \n               <!-- Additional cache configuration parameters -->\n           </bean>\n       </list>\n    </property>\n    \n    <!-- The rest of the configuration. -->\n    <!-- ....... -->\n</bean>  ",
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
Once Apache Ignite cluster is started with this configuration, the page memory will allocate 500 MB memory region in addition to the default one and all the data as well as possible indexes, omitted in this example, of `SampleCache` will reside in that region. The rest of the caches you might have in your deployment will be bound to the default memory region unless you map them to another memory regions explicitly using the technique shown above.
[block:callout]
{
  "type": "success",
  "title": "Changing Default Memory Region",
  "body": "The default memory region is instantiated with the parameters of the memory policy prepared by `org.apache.ignite.configuration.MemoryConfiguration.createDefaultPolicyConfig()` method. If you need to change some parameters of the default region then follow the steps below:\n* Create a new memory policy with a custom name and parameters.\n* Pass the name of the policy to `org.apache.ignite.configuration.MemoryConfiguration.\nsetDefaultMemoryPolicyName(...)` method."
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
    "1-0": "`setSize(...)`",
    "1-1": "Sets maximum memory region size defined by this memory policy.\n\nThe total size can not be less than 1 MB (TODO: double check) due to internal requirements.\n\nIf the overall memory usage goes beyond this parameter an out of memory exception will be thrown. To avoid this use an eviction algorithm or set this parameter to a bigger value.",
    "1-2": "Required parameter.",
    "2-0": "`setSwapFilePath(...)`",
    "2-1": "A path to the memory-mapped file the memory region defined by this memory policy will be mapped to. Having this path set, allows relying on swapping capabilities of an underlying operating system for the memory region.",
    "2-2": "Disabled by default.",
    "3-0": "`setPageEvictionMode(...)`",
    "3-1": "Sets one of the data pages eviction algorithms available for usage. Refer to [eviction policies](doc:evictions) for more details.",
    "3-2": "Disabled by default.",
    "4-0": "`setEvictionThreshold(...)`",
    "4-1": "A threshold for memory pages eviction initiation. For instance, if the threshold is 0.9 it means that the page memory will start the eviction only after 90% of the memory region (defined by this policy) is occupied.",
    "4-2": "`0.9`",
    "5-0": "`setEmptyPagesPoolSize(...)`",
    "5-1": "TODO",
    "5-2": "`100`"
  },
  "cols": 3,
  "rows": 6
}
[/block]