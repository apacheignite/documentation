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
- Combined size of all cache entries (primary and backup) stored on this node. The size of an entry depends on many factors and is not the same as you expect for simple Java objects (see the table below).

## Cache memory usage

Every cache instance takes additional overhead of sum:
- About 2 Mb of basic internal cache objects;
- 8 Mb for internal entry storage for default value `CacheConfiguration.getStartSize()`
Memory consumption could be decreased on smaller cache sizes by adjusting this value, see the table below;
- About 10 Mb for atomic cache queue delete history controlled by the system property `IgniteSystemProperties.IGNITE_ATOMIC_CACHE_DELETE_HISTORY_SIZE`.
[block:parameters]
{
  "data": {
    "h-0": "Cache start size",
    "h-1": "64-bit JVM +UseCompressedOops",
    "h-2": "64-bit JVM -UseCompressedOops",
    "0-0": "< 64k",
    "5-0": "1M...2M",
    "5-1": "8 Mb",
    "5-2": "16 Mb",
    "4-0": "512k...1M",
    "4-1": "4 Mb",
    "4-2": "8 Mb",
    "3-0": "256k...512k",
    "3-1": "2 Mb",
    "3-2": "4 Mb",
    "2-0": "128k...256k",
    "2-1": "1 Mb",
    "2-2": "2 Mb",
    "1-0": "64k...128k",
    "1-1": "512 kb",
    "1-2": "1 Mb",
    "0-1": "< 256 kb",
    "0-2": "< 512 kb"
  },
  "cols": 3,
  "rows": 6
}
[/block]
## Entry memory usage

Actual entry memory usage depends on many factors such as JVM implementation and startup parameters, marshaller implementation, cache atomicity and memory mode. And certainly the key and the value objects itself.
Following calculations have been done for the most common case: Oracle HotSpot Server JVM 64-bit, Binary Marshaller, `ATOMIC` cache mode, and very simple key and value objects.
[block:code]
{
  "codes": [
    {
      "code": "private static class CacheKey {\n\n  public long value;\n\n// POJO\n// 64-bit JVM +UseCompressedOops - 24 bytes\n// 64-bit JVM -UseCompressedOops - 24 bytes\n\n// BinaryMarshaller output byte[34]\n// 64-bit JVM +UseCompressedOops - 56 bytes\n// 64-bit JVM -UseCompressedOops - 56 bytes\n}\n\nprivate static class CacheValue {\n\n  public long value;\n\n  public Object obj = null;\n\n// POJO\n// 64-bit JVM +UseCompressedOops - 24 bytes\n// 64-bit JVM -UseCompressedOops - 32 bytes\n\n// BinaryMarshaller output byte[40]\n// 64-bit JVM +UseCompressedOops - 56 bytes\n// 64-bit JVM -UseCompressedOops - 64 bytes}",
      "language": "java",
      "name": "Example key and value"
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
    "h-2": "Entry overhead",
    "h-3": "First index overhead",
    "1-0": "ONHEAP_TIERED +UseCompressedOops",
    "2-0": "ONHEAP_TIERED -UseCompressedOops",
    "3-0": "ONHEAP_TIERED with off-heap indices",
    "4-0": "OFFHEAP_VALUES",
    "5-0": "OFFHEAP_TIERED",
    "1-1": "320",
    "1-2": "-",
    "1-3": "140",
    "2-1": "490",
    "2-2": "-",
    "2-3": "230",
    "3-1": "320",
    "3-2": "0",
    "3-3": "0",
    "4-1": "0",
    "4-2": "230",
    "4-3": "-",
    "5-1": "0",
    "5-2": "170",
    "5-3": "0",
    "h-5": "Next index overhead",
    "1-5": "40",
    "2-5": "90",
    "3-5": "0",
    "4-5": "-",
    "5-5": "0",
    "0-1": "On-Heap",
    "0-2": "Off-Heap",
    "0-3": "On-Heap",
    "0-4": "Off-Heap",
    "0-5": "On-Heap",
    "0-6": "Off-Heap",
    "1-4": "-",
    "2-4": "-",
    "1-6": "-",
    "2-6": "-",
    "3-4": "270",
    "3-6": "90",
    "4-4": "-",
    "4-6": "-",
    "5-4": "230",
    "5-6": "70",
    "h-4": "First index overhead",
    "h-6": "Next index overhead"
  },
  "cols": 7,
  "rows": 6
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

- Cache memory usage = 2 Mb + internal storage + delete history;
Internal storage = cache size 4M...8M = 32 Mb;
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