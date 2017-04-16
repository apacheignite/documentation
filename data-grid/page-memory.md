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

* On-heap tier: data stored in this memory layer resided in the Java heap managed by Java VM. If the overall data set's size grew beyond 20 GB on a single Apache Ignite node then you could easily face long stop-the-wold garbage collection pauses that can be a significant performance hit or cause complete ejection of the node from the cluster. Considering that       

Page Memory is an abstraction for working with pages in memory. Internally it interacts with a file store which is responsible for allocating page IDs, writing and reading of pages. One should distinguish a concept of a page and a page buffer.

A page is a block of data of a fixed length with a unique identifier called FullPageId. 


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