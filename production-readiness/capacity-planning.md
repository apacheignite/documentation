* [Calculating the Memory Usage](doc:capacity-planning#calculating-the-memory-usage)
* [Capacity Planning Example](doc:capacity-planning#capacity-planning-example)

When preparing and planning for a system capacity planning is an integral part of the design. Understanding the size of the cache that will be required will help decide how much physical memory, how many JVMs, and how many CPUs and servers will be required. In this section we discuss various techniques that can help plan and identify the minimum hardware requirements for a given deployment.
[block:api-header]
{
  "type": "basic",
  "title": "Calculating the Memory Usage"
}
[/block]
- Calculate primary data size: multiply the size of one entry in bytes by the total number of entries
- If you have backups, multiply by their number
- Indexes also require memory. Basic use cases will add a 30% increase
- Add around 20MB per cache. This value can be reduced if to decrease partitions count and/or IgniteSystemProperties.IGNITE_ATOMIC_CACHE_DELETE_HISTORY_SIZE
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