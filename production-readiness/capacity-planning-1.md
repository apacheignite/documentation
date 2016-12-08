* [Overview](#overview)
* [Calculating Memory Usage](#calculating-memory-usage)
* [Calculating Compute Usage](#calculating-compute-usage)
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
Memory usage for every JVM with running Ignite instances is made with following parts:
- Minimum 150 Mb for internal memory of the JVM with common Ignite overhead;
- About 50 Mb per every running Ignite instance under the JVM;
- About 20 Mb per cache (this size could be decreased, see instructions below);
- Combined size of all cache entries (primary and backup) stored on this node; the size of an entry depends on many factors and is not the same as you expect for simple Java objects (see the table below).
[block:callout]
{
  "type": "warning",
  "body": "When calculating number of partitioned cache entries may be kept on a node, take into account that the affinity function may not distribute all entries among nodes equally.",
  "title": ""
}
[/block]


## Memory Capacity Planning Example

Let's take for example the following scenario:
[block:callout]
{
  "type": "success",
  "body": "- 2,000,000 objects\n- 1,024 bytes per object (1 KB)\n- 1 backup\n- 4 nodes",
  "title": "Example Specification"
}
[/block]
- Total number of objects X object size X 2 (one primary and one backup copy for each object):
2,000,000 x 1,024 x 2 = 4,096,000,000 bytes

- Considering indexes:
4,096,000,000 + (4,096,000,000 x 30%) = 5,078 MB

- Approximate additional memory required by the platform:
300 MB x 4 = 1,200 MB

- Total size:
5,078 + 1,200 = 6,278 MB

Hence the anticipated total memory consumption would be just over ~ 6 GB
[block:api-header]
{
  "type": "basic",
  "title": "Calculating Compute Usage"
}
[/block]
Calculating compute is generally much harder to estimate without some code already in place. It is important to understand the cost of a given operation that your application will be performing and multiply this by the number of operations expected at various times. A good starting point for this would be the Ignite benchmarks which detail the results of standard operations and give a rough estimate of the capacity required to deliver such performance.

With 32 cores over 4 large AWS instances the following benchmarks were recorded:
[block:callout]
{
  "type": "success",
  "body": "- PUT/GET: 26k/sec\n- PUT (TRANSACTIONAL): 68k/sec\n- PUT (TRANSACTIONAL - PESSIMISTIC): 20k/sec\n- PUT (TRANSACTIONAL - OPTIMISTIC): 44k/sec\n- SQL Query: 72k/sec"
}
[/block]
[More results here](http://www.gridgain.com/resources/benchmarks/ignite-vs-hazelcast-benchmarks)
[block:api-header]
{
  "type": "basic",
  "title": "Capacity Planning FAQ"
}
[/block]
**- I have 300GB of data in DB will this be the same in Ignite?**

No, data size on disk is not a direct 1-to-1 mapping in memory. As a very rough estimate it can be about 2.5/3 times size on disk excluding indexes and any other overhead. To get a more accurate estimation you need to figure out the average object size by importing a record into Ignite and multiplying by the number of objects expected.