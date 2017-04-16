* [Overview](doc:page-memory#overview)
* [Page Memory](doc:page-memory#page-memory)
* [Data Page](doc:page-memory#data-page) 
* [Index Page](doc:page-memory#index-page)
* [Page Buffer](doc:page-memory#page-buffer)
* [Configuration](doc:page-memory#configuration)
* [Memory Policies](doc:page-memory#memory-policies)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Prior to Apache Ignite 2.0, the data grid's distributed key-value store was built on top and relied on three memory tiers - on-heap tier, off-heap tier, and swap tier. All these memory modes had some tier-specific disadvantages:

* On-heap tier: data stored in this memory layer resided in the Java heap managed by Java VM. If the overall data set's size grew beyond 20 GB on a single Apache Ignite node then you could easily face long stop-the-world garbage collection pauses that in return could cause a significant performance hit or complete ejection of the node from the cluster. Considering that in most modern Apache Ignite use cases the overall memory consumption on individual nodes goes far beyond than 20 GB the off-heap tier was started to be used as a default memory mode to mitigate these on-heap tier issues.

* Off-heap tier: data was stored in a manageable memory region outside of Java heap (off-heap). All the memory allocations and releases were managed by Apache Ignite internally in a fully transparent fashion for the end user. This memory mode helped to resolve all the on-heap tier's issues and allowed to store hundreds of gigabytes and petabytes of data on an individual node effortlessly. However, the off-heap tier still had some disadvantages on its own such as uncontrollable memory fragmentation and necessity to maintain SQL rows/indexes cache in Java heap that barred from doing some of the most valuable SQL Grid optimization tasks.

* Swap tier: when there was no more room left in one of the two tiers above, the data could have been swapped to disk to the Apache Ignite swap tier. Frequent access to the swapped tier could cause significant performance degradation of the overall application especially if SQL queries were used because considerable amount of data might have been moved between the swap and other memory layers.          

All the memory tiers listed above have been discontinued in Apache Ignite 2.0 in favor of a completely new off-heap based `page memory` that has the following benefits:
* No stop-the-world (STW) garbage collection (GC) pauses caused by Apache Ignite platform architecture. Application's code is the only one possible source of STW GC pauses.
* No memory fragmentation.
* Profoundly improves the performance of and memory utilization by SQL Grid and opens up a room for an enormous number of further optimizations.
* Transparent swapping: the page memory can be mapped to a memory-mapped file offloading swapping related activity to an underlying operating system.
* An ability to integrate with a file system in order to persist memory pages on disk for durability purpose and create cluster-wide snapshots (backups).
[block:api-header]
{
  "type": "basic",
  "title": "Data Page"
}
[/block]
TBD
[block:api-header]
{
  "type": "basic",
  "title": "Index Page"
}
[/block]
TBD
[block:api-header]
{
  "type": "basic",
  "title": "Page Buffer"
}
[/block]
A page buffer is a region in memory associated with some page.

Page Memory fully handles the process of loading pages to the corresponding page buffers and evicting unused page buffers from memory. At any moment in time page memory may keep any subset of page buffers (fitting to the allocated RAM).

## Page Allocation

TBD (What happens when the persistent store is enabled and disabled?)

What happens when the persistent store is enabled and disabled?

## Page Eviction

TBD (What happens when the persistent store is enabled and disabled?)

## Primary vs Backup Data

TBD (It's known that a node will first keep primary data in RAM rather than backup data. We need to describe this algorithm​ here). 
[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
## Page Memory Configuration Parameters

TBD

## Page Configuration Parameters

TBD

## Page Buffer Configuration Parameters

TBD
[block:api-header]
{
  "type": "basic",
  "title": "Memory Policies"
}
[/block]
TBD