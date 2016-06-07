* [Overview](#overview)
* [IgniteDataStreamer](#ignitedatastreamer)
* [IgniteCache.loadCache()](#ignitecacheloadcache)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Data loading usually has to do with initializing cache data on startup. Using standard cache `put(...)` or `putAll(...)` operations is generally inefficient for loading large amounts of data. Ignite offers multiple ways to to load large amounts of data into Ignite caches.
[block:api-header]
{
  "type": "basic",
  "title": "IgniteDataStreamer"
}
[/block]
Data streamers are defined by `IgniteDataStreamer` API and are built to inject large amounts of continuous data into Ignite caches. Data streamers are built in a scalable and fault-tolerant fashion and achieve high performance by batching entries together before they are sent to the corresponding cluster members.
[block:callout]
{
  "type": "success",
  "body": "Data streamers should be used to load large amount of data into caches at any time, including pre-loading on startup."
}
[/block]
See  [Data Streamers](doc:data-streamers) documentation for more information.
[block:api-header]
{
  "type": "basic",
  "title": "IgniteCache.loadCache()"
}
[/block]
Another way to load large amounts of data into cache is through [CacheStore.loadcache()](doc:persistent-store#section-loadcache-) method, which allows for cache data loading even without passing all the keys that need to be loaded. 

`IgniteCache.loadCache()` method will delegate to `CacheStore.loadCache()` method on every cluster member that is running the cache. To invoke loading only on the local cluster node, use `IgniteCache.localLoadCache()` method.
[block:callout]
{
  "type": "info",
  "body": "In case of partitioned caches, keys that are not mapped to this node, either as primary or backups, will be automatically discarded by the cache."
}
[/block]
Here is an example of how `CacheStore.loadCache()` implementation. For a complete example of how a `CacheStore` can be implemented refer to [Persistent Store](doc:persistent-store).
[block:code]
{
  "codes": [
    {
      "code": "public class CacheJdbcPersonStore extends CacheStoreAdapter<Long, Person> {\n\t...\n  // This method is called whenever \"IgniteCache.loadCache()\" or\n  // \"IgniteCache.localLoadCache()\" methods are called.\n  @Override public void loadCache(IgniteBiInClosure<Long, Person> clo, Object... args) {\n    if (args == null || args.length == 0 || args[0] == null)\n      throw new CacheLoaderException(\"Expected entry count parameter is not provided.\");\n\n    final int entryCnt = (Integer)args[0];\n\n    Connection conn = null;\n\n    try (Connection conn = connection()) {\n      try (PreparedStatement st = conn.prepareStatement(\"select * from PERSONS\")) {\n        try (ResultSet rs = st.executeQuery()) {\n          int cnt = 0;\n\n          while (cnt < entryCnt && rs.next()) {\n            Person person = new Person(rs.getLong(1), rs.getString(2), rs.getString(3));\n\n            clo.apply(person.getId(), person);\n\n            cnt++;\n          }\n        }\n      }\n    }\n    catch (SQLException e) {\n      throw new CacheLoaderException(\"Failed to load values from cache store.\", e);\n    }\n  }\n  ...\n}",
      "language": "java"
    }
  ]
}
[/block]
## Partition-aware data loading

In the scenario described above the same query will be executed on all the nodes. Each node will iterate over the whole result set, skipping the keys that do not belong to the node, which is not very efficient. 

The situation may be improved if partition ID is stored alongside with each record in the database. You can use `org.apache.ignite.cache.affinity.Affinity` interface to get partition ID for any key being stored into a cache.

Below is an example code snippet that determines partition ID for each `Person` object being stored into the cache.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache cache = ignite.cache(cacheName);\nAffinity aff = ignite.affinity(cacheName);\n\nfor (int personId = 0; personId < PERSONS_CNT; personId++) {\n    // Get partition ID for the key under which person is stored in cache.\n    int partId = aff.partition(personId);\n  \n    Person person = new Person(personId);\n    person.setPartitionId(partId);\n    // Fill other fields.\n  \n    cache.put(personId, person);\n}",
      "language": "java"
    }
  ]
}
[/block]
When `Person` objects become partition-ID aware, each node can query only those partitions that belong to the node. In order to do that, you can inject an instance of Ignite into your cache store and use it to determine partitions that belong to the local node. 

Below is an example code snippet that demonstrates how to use `Affinity` to load only local partitions. Note that example code is single-threaded, however it can be very effectively parallelized by partition ID.
[block:code]
{
  "codes": [
    {
      "code": "public class CacheJdbcPersonStore extends CacheStoreAdapter<Long, Person> {\n  // Will be automatically injected.\n  @IgniteInstanceResource\n  private Ignite ignite;\n  \n\t...\n  // This mehtod is called whenever \"IgniteCache.loadCache()\" or\n  // \"IgniteCache.localLoadCache()\" methods are called.\n  @Override public void loadCache(IgniteBiInClosure<Long, Person> clo, Object... args) {\n    Affinity aff = ignite.affinity(cacheName);\n    ClusterNode locNode = ignite.cluster().localNode();\n    \n    try (Connection conn = connection()) {\n      for (int part : aff.primaryPartitions(locNode))\n        loadPartition(conn, part, clo);\n      \n      for (int part : aff.backupPartitions(locNode))\n        loadPartition(conn, part, clo);\n    }\n  }\n  \n  private void loadPartition(Connection conn, int part, IgniteBiInClosure<Long, Person> clo) {\n    try (PreparedStatement st = conn.prepareStatement(\"select * from PERSONS where partId=?\")) {\n      st.setInt(1, part);\n      \n      try (ResultSet rs = st.executeQuery()) {\n        while (rs.next()) {\n          Person person = new Person(rs.getLong(1), rs.getString(2), rs.getString(3));\n          \n          clo.apply(person.getId(), person);\n        }\n      }\n    }\n    catch (SQLException e) {\n      throw new CacheLoaderException(\"Failed to load values from cache store.\", e);\n    }\n  }\n  \n  ...\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Note that key-to-partition mapping depends on the number of partitions configured in the affinity function (see `org.apache.ignite.cache.affinity.AffinityFunction`). If affinity function configuration changes, partition ID records in the database must be updated accordingly."
}
[/block]