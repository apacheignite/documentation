--------------
title: Data Loading
excerpt: Load large amounts of data into cache.
--------------

Data loading usually has to do with initializing cache data on startup. Using standard cache `put(...)` or `putAll(...)` operations is generally inefficient for loading large amounts of data. 
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
Another way to load large amounts of data into cache is through [CacheStore.loadCache()](docs/persistent-store#loadcache-) method, which allows for cache data loading even without passing all the keys that need to be loaded. 

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
      "code": "public class CacheJdbcPersonStore extends CacheStoreAdapter<Long, Person> {\n\t...\n  // This mehtod is called whenever \"IgniteCache.loadCache()\" or\n  // \"IgniteCache.localLoadCache()\" methods are called.\n  @Override public void loadCache(IgniteBiInClosure<Long, Person> clo, Object... args) {\n    if (args == null || args.length == 0 || args[0] == null)\n      throw new CacheLoaderException(\"Expected entry count parameter is not provided.\");\n\n    final int entryCnt = (Integer)args[0];\n\n    Connection conn = null;\n\n    try (Connection conn = connection()) {\n      try (PreparedStatement st = conn.prepareStatement(\"select * from PERSONS\")) {\n        try (ResultSet rs = st.executeQuery()) {\n          int cnt = 0;\n\n          while (cnt < entryCnt && rs.next()) {\n            Person person = new Person(rs.getLong(1), rs.getString(2), rs.getString(3));\n\n            clo.apply(person.getId(), person);\n\n            cnt++;\n          }\n        }\n      }\n    }\n    catch (SQLException e) {\n      throw new CacheLoaderException(\"Failed to load values from cache store.\", e);\n    }\n  }\n  ...\n}",
      "language": "java"
    }
  ]
}
[/block]