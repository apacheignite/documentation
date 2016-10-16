A partitioned cache can also be fronted by a `Near` cache, which is a smaller local cache that stores most recently or most frequently accessed data. Just like with a partitioned cache, the user can control the size of the near cache and its eviction policies. 

Near caches can be created directly on *client* nodes by passing `NearConfiguration` into the `Ignite.createNearCache(NearConfiguration)` or `Ignite.getOrCreateNearCache(NearConfiguration)` methods.
[block:code]
{
  "codes": [
    {
      "code": "// Create near-cache configuration for \"myCache\".\nNearCacheConfiguration<Integer, Integer> nearCfg = \n    new NearCacheConfiguration<>();\n\n// Use LRU eviction policy to automatically evict entries\n// from near-cache, whenever it reaches 100_000 in size.\nnearCfg.setNearEvictionPolicy(new LruEvictionPolicy<>(100_000));\n\n// Create a distributed cache on server nodes and \n// a near cache on the local node, named \"myCache\".\nIgniteCache<Integer, Integer> cache = ignite.getOrCreateCache(\n    new CacheConfiguration<Integer, Integer>(\"myCache\"), nearCfg);",
      "language": "java"
    }
  ]
}
[/block]
In the vast majority of use cases, whenever utilizing Ignite with affinity colocation, near caches should not be used. If computations are collocated with the corresponding partition cache nodes then the near cache is simply not needed because all the data is available locally in the partitioned cache.

However, there are cases when it is simply impossible to send computations to remote nodes. For cases like this near caches can significantly improve scalability and the overall performance of the application.
[block:callout]
{
  "type": "info",
  "title": "Transactions",
  "body": "Near caches are fully transactional and get updated or invalidated automatically whenever the data changes on the servers."
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Near Caches on Server Nodes",
  "body": "Whenever accessing data from `PARTITIONED` caches on the server side in a non-collocated fashion, you may need to configure near-caches on the server nodes via `CacheConfiguration.setNearConfiguration(...)` property."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
Most configuration parameters available on `CacheConfiguration` that make sense for the near cache are inherited from the server configuration. For example, if the server cache has an `ExpiryPolicy`, entries in the near cache will be expired based on the same policy.

Parameters listed in the table below are not inherited from the server configuration and provided separately, via the `NearCacheConfiguration` object.
[block:parameters]
{
  "data": {
    "0-0": "`setNearEvictionPolicy(CacheEvictionPolicy)`",
    "h-0": "Setter Method",
    "h-1": "Description",
    "h-2": "Default",
    "0-1": "Eviction policy for the near cache.",
    "0-2": "None",
    "1-0": "`setNearStartSize(int)`",
    "1-2": "`375,000`",
    "1-1": "Eviction policy for near cache."
  },
  "cols": 3,
  "rows": 2
}
[/block]