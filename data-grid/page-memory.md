* [Page Memory](doc:page-memory#page-memory)
* [Page](doc:page-memory#page)
* [Data Page](doc:page-memory#data-page) 
* [Index Page](doc:page-memory#index-page)
* [Page Buffer](doc:page-memory#page-buffer)
* [Configuration](doc:page-memory#configuration)
* [Memory Policies](doc:page-memory#memory-policies)
* [Page Memory Internals](doc:page-memory#page-memory-internals)
[block:api-header]
{
  "type": "basic",
  "title": "Page Memory"
}
[/block]
Page Memory is an abstraction for working with pages in memory. Internally it interacts with a file store which is responsible for allocating page IDs, writing and reading of pages. One should distinguish a concept of a page and a page buffer.
[block:api-header]
{
  "type": "basic",
  "title": "Page"
}
[/block]
A page is a block of data of a fixed length with a unique identifier called FullPageId. 


## FullPageId 

FullPageId consists of cache ID (32 bits) and page ID (64 bits). Page ID is effectively a virtual page identifier that may change during the page lifecycle (see PageRotation below). EffectivePageId, which is a part of Page ID (partition ID, page index) does not change during the page lifecycle.

## PageId and EffectivePageId

The structure of page ID is defined by the following diagram:
+-------------+-------------+---------------------+--------------------------+
| 8 bits        |     8 bits   |       16 bits         |         32 bits            |
+-------------+-------------+---------------------+--------------------------+
| OFFSET  |    FLAGS |  PARTITION ID |        PAGE INDEX   |
+-------------+-------------+---------------------+--------------------------+
                                    |            EFFECTIVE PAGE ID            |
                                   +-------------------------------------------------+
* Offset is a reserved field used either for page ID rotation or for referencing a record in a data page.
* Flags are a reserved field used for page ID rotation.
* Partition ID is either a partition identifier in [0, 65500] or a reserved value 0xFFFF used for index partition. Other values are reserved for future use.
* Page Index is a monotonically growing number within each partition.

## Page State

At any moment in time page can be in the following states:
* Unloaded. There is no a corresponding page buffer loaded in memory.
* Clean. Page buffer is loaded and page buffer content is equal to the data written to disk.
* Dirty. Page buffer has been modified and it's content is different from the data written to disk.
* Dirty in checkpoint. Page buffer has been modified, checkpoint started and page buffer has been modified again before the first modification has been written to disk. In this state PageMemory keeps two page buffers for each page - first one is for the current checkpoint (in progress) and the second one is for the next checkpoint.

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
[block:api-header]
{
  "type": "basic",
  "title": "Page Memory Internals"
}
[/block]
## Page Modification

Each page buffer has an associated read write lock paired with a tag. Page tag is a part of page ID used for page rotation. In order to read or write page buffer contents one must acquire the read or write lock. The page buffer read write lock requires a correct tag to be passed in order for the lock to be acquired. If the passed in tag differs from the actual page tag, it means that the page has been reused and the link to this page is no longer valid (see page ID rotation below).

## Internal Data Structures

Depending on page type and the amount of free space in a page it can be optionally tracked in either FreeList or ReuseList. FreeList is a data structure used to track partially free pages (applicable for data pages). ReuseList tracks completely free pages that can be used by any other data structure in the database. 

Each partition has a special dedicated page (a meta page) which holds the state of the partition and page IDs serving as roots for all the data structures associated with the partition.

## Data Partitions

For data partitions, the following data structures are used:
* FreeList (working as a reuse list at the same time) to track free and empty pages.
* Partition hash index used as a main storage for the cache.
* RowStore to keep key-value pairs.

## Index Partition

For index partition, the following data structures are used:
* ReuseList (since BTrees fully acquire pages and do not need to track free space)
* Metadata storage. This data structure keeps track of all allocated indexes and contains pairs (index name, root page ID)