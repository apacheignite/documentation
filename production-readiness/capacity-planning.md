* [Overview](#overview)
* [Calculating Memory Usage](#calculating-memory-usage)
* [Capacity Planning Example](#capacity-planning-example)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
When preparing and planning for a system capacity planning is an integral part of the design. Understanding the size of the cache that will be required will help decide how much physical memory, how many JVMs, and how many CPUs and servers will be required. In this section we discuss various techniques that can help plan and identify the minimum hardware requirements for a given deployment.
[block:api-header]
{
  "type": "basic",
  "title": "Calculating Memory Usage"
}
[/block]
- Calculate primary data size: multiply the size of one entry in bytes by the total number of entries
- If you have backups, multiply by their number
- Indexes also require memory. Basic use cases will add a 30% increase
- Add around 20MB per cache. (This value can be reduced if you explicitly set `IgniteSystemProperties.IGNITE_ATOMIC_CACHE_DELETE_HISTORY_SIZE` to a smaller value than default.)
- Add around 200-300MB per node for internal memory and reasonable amount of memory for JVM and GC to operate efficiently
[block:callout]
{
  "type": "info",
  "body": "GridGain will typically add around 200 bytes overhead to each entry."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Capacity Planning Example"
}
[/block]
Let's take for example the following scenario:

- 2,000,000 objects
- 1,024 bytes per object (1 KB)
- 1 backup
- 4 nodes

Total number of objects X object size X 2 (one primary and one backup copy for each object):
2,000,000 x 1,024 x 2 = 4,096,000,000 bytes

Considering indexes:
4,096,000,000 + (4,096,000,000 x 30%) = 5,078 MB

Approximate additional memory required by the platform
300 MB x 4 = 1,200 MB

Total size
5,078 + 1,200 = 6,278 MB

Hence the anticipated total memory consumption would be just over ~ 6 GB
[block:api-header]
{
  "type": "basic",
  "title": "Capacity Planning FAQ"
}
[/block]
**- I have 300GB of data in DB will this be the same in GridGain?**

No, data size on disk is not a direct 1-to-1 mapping in memory. As a very rough estimate it can be about 2.5/3 times size on disk excluding indexes and any other overhead. To get a more accurate estimation you need to figure out the average object size by importing a record into GridGain and multiplying by the number of objects expected.

**- How do I plan for processing capacity?**

This is generally much harder to estimate without some code already in place. It is important to understand the cost of a given operation that your application will be performing and multiply this by the number of operations expected at various times. A good starting point for this would be the Ignite benchmarks which detail the results of standard operations and give a rough estimate of the capacity required to deliver such performance.
[block:callout]
{
  "type": "info",
  "title": "Processing Benchmarks",
  "body": "With 32 cores over 4 large AWS instances the following benchmarks were recorded:\n- **PUT/GET: 26k/sec**\n- **PUT (TRANSACTIONAL): 68k/sec**\n- **PUT (TRANSACTIONAL - PESSIMISTIC): 20k/sec**\n- **PUT (TRANSACTIONAL - OPTIMISTIC): 44k/sec**\n- **SQL Query: 72k/sec**\n[More results here](http://www.gridgain.com/resources/benchmarks/ignite-vs-hazelcast-benchmarks)"
}
[/block]