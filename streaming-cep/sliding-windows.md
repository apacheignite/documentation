Sliding windows are configured as Ignite cache eviction policies, and can be time-based, size-based, or batch-based. You can configure one sliding-window per cache. However, you can easily define more than one cache if you need different sliding windows for the same data.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/cZwoaIpoRyygc8BJFRJL_in_memory_streaming.png",
        "in_memory_streaming.png",
        "600",
        "180",
        "#e45e6d",
        ""
      ]
    }
  ]
}
[/block]
##Time-Based Sliding Windows
Time-based windows are configured using JCache-compliant `ExpiryPolicy`. You can have streamed events expire based on **create time**, **last-access time**, or **update time**.

Here is how you can configure a 5-second sliding window based on creation time in Ignite.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<Integer, Long> cfg = new CacheConfiguration<>(\"myStreamCache\");\n\n// Sliding window of 5 seconds based on creation time.\ncfg.setExpiryPolicyFactory(FactoryBuilder.factoryOf(\n  new CreatedExpiryPolicy(new Duration(SECONDS, 5))));",
      "language": "java"
    }
  ]
}
[/block]
##FIFO Sliding Window
FIFO (first-in-first-out) sliding windows are configured via `FifoEvictionPolicy` in Ignite caches. This policy is size-based. Stream tuples are inserted into the window until cache size reaches its maximum limit. Then the oldest tuples start getting evicted automatically. 

Here is how you can configure a FIFO sliding window holding 1,000,000 of stream tuples.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<Integer, Long> cfg = new CacheConfiguration<>(\"myStreamCache\");\n\n// FIFO window holding 1,000,000 entries.\ncfg.setEvictionPolicyFactory(new FifoEvictionPolicy(1_000_000));",
      "language": "java"
    }
  ]
}
[/block]
##LRU Sliding Window
LRU (least-recently-used) sliding windows are configured via `LruEvictionPolicy` in Ignite caches. This policy is size-based. Stream tuples are inserted into the window until cache size reaches its maximum limit. Then the least recently accessed tuples start getting evicted automatically. 

Here is how you can configure LRU sliding window holding 1,000,000 of most recently accessed data.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<Integer, Long> cfg = new CacheConfiguration<>(\"myStreamCache\");\n\n// LRU window holding 1,000,000 entries.\ncfg.setEvictionPolicyFactory(new LruEvictionPolicy(1_000_000));",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Querying Sliding Windows"
}
[/block]
Sliding windows can be queried in the same way as any other Ignite caches, using Predicate-based, SQL, or TEXT queries. 

Here is an example of how a cache holding a sliding window of financial instruments streamed into the system can be queried using SQL queries.

First we need to create indexes based on the queried fields. In this case we are indexing the fields for *Instrument* class and the *String* keys.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<String, Instrument> cfg = new CacheConfiguration<>(\"instCache\");\n\n// Index some fields for querying portfolio positions.\ncfg.setIndexedTypes(String.class, Instrument.class);\n\n// Get a handle on the cache (create it if necessary).\nIgniteCache<String, Instrument> instCache = ignite.getOrCreateCache(cfg);",
      "language": "java"
    }
  ]
}
[/block]
Let's query top 3 best performing financial instruments. We do it via ordering by *(latest - open)* price and selecting top 3.
[block:code]
{
  "codes": [
    {
      "code": "// Select top 3 best performing instruments.\nSqlFieldsQuery top3qry = new SqlFieldsQuery(\n  \"select symbol, (latest - open) from Instrument order by (latest - open) desc limit 3\");\n\n// List of rows. Every row is represented as a List as well.\nList<List<?>> top3 = instCache.query(top3qry).getAll();",
      "language": "java"
    }
  ]
}
[/block]
Let's query total profit across all financial instruments. We do it by adding up all *(latest - open)* values across all instruments.
[block:code]
{
  "codes": [
    {
      "code": "// Select total profit across all financial instruments.\nSqlFieldsQuery profitQry = new SqlFieldsQuery(\"select sum(latest - open) from Instrument\");\n\nList<List<?>> profit = instCache.query(profitQry).getAll();\n\nSystem.out.printf(\"Total profit: %.2f%n\", row.get(0));",
      "language": "java"
    }
  ]
}
[/block]