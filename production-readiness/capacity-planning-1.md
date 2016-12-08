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
## Cache memory usage

- About 2 Mb of basic internal cache objects;
- 8 Mb for internal entry storage for default `CacheConfiguration.getStartSize()` value approximated to the nearest bigger value of power of two; see the formula below:
**startSize** - the value of `CacheConfiguration.getStartSize()`;
**partNumber** - the value of `AffinityFunction.partitions()`;
**refSize** - the size of the Java reference (4 bytes on 32-bit JVM and 64-bit w/UseCompressedOops enabled, or 8 bytes on 64-bit JVM w/UseCompressedOops disabled);
**partSize** = 2 ^ roundup( log_2( startSize / partNumber ) );
**overhead** = partSize x partNumber x refSize.
For example, default startSize = 1'500'000, partNumber = 1024, refSize = 4, partSize = 2048 (the nearest bigger 2^N for 1'500'000/1024), overhead = 2K x 1K x 4 = 8 Mb.
- About 10 Mb for atomic cache queue delete history controlled by the system property `IgniteSystemProperties.IGNITE_ATOMIC_CACHE_DELETE_HISTORY_SIZE`.

## Entry memory usage

Actual entry memory usage depends on many factors such as JVM implementation and startup parameters, marshaller implementation, cache atomicity and memory mode. And certainly the key and the value objects itself.
Following calculations have been done for the most common case: Oracle HotSpot Server JVM 64-bit, Binary Marshaller, `ATOMIC` cache mode, and very simple key and value objects.
[block:code]
{
  "codes": [
    {
      "code": "private static class CacheKey {\n\n  public long value;\n}\n\n// POJO\n// 32-bit JVM - 16 bytes\n// 64-bit JVM +UseCompressedOops - 24 bytes\n// 64-bit JVM -UseCompressedOops - 24 bytes\n\n// BinaryMarshaller output byte[34]\n// 32-bit JVM - 48 bytes\n// 64-bit JVM +UseCompressedOops - 56 bytes\n// 64-bit JVM -UseCompressedOops - 56 bytes",
      "language": "java",
      "name": "CacheKey"
    },
    {
      "code": "private static class CacheValue {\n\n  public long value;\n\n  public Object obj = null;\n}\n\n// 32-bit JVM - 20 bytes\n// 64-bit JVM +UseCompressedOops - 24 bytes\n// 64-bit JVM -UseCompressedOops - 32 bytes\n\n// BinaryMarshaller output byte[40]\n// 32-bit JVM - 52 bytes\n// 64-bit JVM +UseCompressedOops - 56 bytes\n// 64-bit JVM -UseCompressedOops - 64 bytes",
      "language": "java",
      "name": "CacheValue"
    }
  ]
}
[/block]
You could see that additional overhead arises from serializing key and value objects. The following table already contains that overhead, but actual value may vary for another objects (it even may be negative for complex Java objects).
[block:parameters]
{
  "data": {
    "h-0": "Cache configuration",
    "h-1": "Entry overhead",
    "h-2": "First index overhead",
    "h-3": "Next index overhead",
    "0-0": "ONHEAP_TIERED +UseCompressedOops",
    "1-0": "ONHEAP_TIERED -UseCompressedOops",
    "2-0": "ONHEAP_TIERED with off-heap indices",
    "3-0": "OFFHEAP_VALUES",
    "4-0": "OFFHEAP_TIERED",
    "0-1": "340",
    "0-2": "140",
    "0-3": "40",
    "1-1": "490",
    "1-2": "230",
    "1-3": "90",
    "2-1": "320",
    "2-2": "270",
    "2-3": "90",
    "3-1": "230",
    "3-2": "-",
    "3-3": "-",
    "4-1": "170",
    "4-2": "230",
    "4-3": "70"
  },
  "cols": 4,
  "rows": 5
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Note, that unlike regular database indices, Ignite indices take constant overhead per entry independently of entry size."
}
[/block]

[block:callout]
{
  "type": "success",
  "body": "- 5,000,000 entries (given CacheKey & CacheValue for example)\n- 868 bytes per object (ONHEAP_TIERED w/UseCompressedOops enabled, one index)\n- 1 backup\n- 4 nodes",
  "title": "Example Specification"
}
[/block]
- Entries memory usage = entry size x number of entries per node x 2 (one primary and one backup copy);
 868 x 1'250'000 x 2 = 2'170'000'000 = 2069 Mb;

- Cache size = 2 Mb + internal storage + delete history;
Internal storage = 8192 x 1024 x 4 = 32 Mb;
Delete history = 10 Mb
2 + 32 + 10 = 44 Mb

- JVM + Ignite instance = 200 Mb

- Total memory usage per node:
2069 + 44 + 200 = 2313 Mb

Hence the anticipated total memory consumption would be just over ~ 9 GB
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