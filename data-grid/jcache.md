## On this page
* [IgniteCache API](doc:jcache#ignitecache)
* [Basic Operations](doc:jcache#basic-operations)
* [EntryProcessor](doc:jcache#entryprocessor)
* [Asynchronous Support](doc:jcache#asynchronous-support)

Apache Ignite data grid is an implementation of **JCache (JSR 107)** specification. JCache provides a very simple to use, but yet very powerful API for data access. However, the specification purposely omits any details about data distribution and consistency to allow vendors enough freedom in their own implementations. 

With JCache support you get the following:
  * Basic Cache Operations
  * ConcurrentMap APIs
  * Collocated Processing (EntryProcessor)
  * Events and Metrics
  * Pluggable Persistence

In addition to JCache, Ignite provides ACID transactions, data querying capabilities (including SQL), various memory models, queries, transactions, etc...
[block:api-header]
{
  "type": "basic",
  "title": "IgniteCache API"
}
[/block]
`IgniteCache` interface is a gateway into Ignite cache implementation and provides methods for storing and retrieving data, executing queries, including SQL, iterating and scanning, etc.

`IgniteCache` is based on **JCache (JSR 107)**, so at the very basic level the APIs can be reduced to `javax.cache.Cache` interface. However, `IgniteCache` API also provides functionality that facilitates features outside of JCache spec, like data loading, querying, asynchronous mode, etc.

You can get an instance of `IgniteCache` directly from `Ignite`:
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\n// Obtain instance of cache named \"myCache\".\n// Note that different caches may have different generics.\nIgniteCache<Integer, String> cache = ignite.cache(\"myCache\");",
      "language": "java"
    }
  ]
}
[/block]
## Dynamic Cache
You can also create an instance of the cache on the fly, in which case Ignite will create and deploy the cache across all server cluster members.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nCacheConfiguration cfg = new CacheConfiguration();\n\ncfg.setName(\"myCache\");\ncfg.setAtomicityMode(TRANSACTIONAL);\n\n// Create cache with given name, if it does not exist.\nIgniteCache<Integer, String> cache = ignite.getOrCreateCache(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "All caches defined in Ignite Spring XML configuration on any cluster member will also be automatically created and deployed on all the cluster servers (no need to specify the same configuration on each cluster member).",
  "title": "XML Configuration"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Basic Operations"
}
[/block]
Here are some basic JCache atomic operation examples.
[block:code]
{
  "codes": [
    {
      "code": "try (Ignite ignite = Ignition.start(\"examples/config/example-cache.xml\")) {\n    IgniteCache<Integer, String> cache = ignite.cache(CACHE_NAME);\n \n    // Store keys in cache (values will end up on different cache nodes).\n    for (int i = 0; i < 10; i++)\n        cache.put(i, Integer.toString(i));\n \n    for (int i = 0; i < 10; i++)\n        System.out.println(\"Got [key=\" + i + \", val=\" + cache.get(i) + ']');\n}",
      "language": "java",
      "name": "Put & Get"
    },
    {
      "code": "// Put-if-absent which returns previous value.\nInteger oldVal = cache.getAndPutIfAbsent(\"Hello\", 11);\n  \n// Put-if-absent which returns boolean success flag.\nboolean success = cache.putIfAbsent(\"World\", 22);\n  \n// Replace-if-exists operation (opposite of getAndPutIfAbsent), returns previous value.\noldVal = cache.getAndReplace(\"Hello\", 11);\n \n// Replace-if-exists operation (opposite of putIfAbsent), returns boolean success flag.\nsuccess = cache.replace(\"World\", 22);\n  \n// Replace-if-matches operation.\nsuccess = cache.replace(\"World\", 2, 22);\n  \n// Remove-if-matches operation.\nsuccess = cache.remove(\"Hello\", 1);",
      "language": "java",
      "name": "Atomic"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "EntryProcessor"
}
[/block]
Whenever doing `puts` and `updates` in cache, you are usually sending full state object state across the network. `EntryProcessor` allows for processing data directly on primary nodes, often transferring only the deltas instead of the full state. 

Moreover, you can embed your own logic into `EntryProcessors`, for example, taking previous cached value and incrementing it by 1.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<String, Integer> cache = ignite.cache(\"mycache\");\n\n// Increment cache value 10 times.\nfor (int i = 0; i < 10; i++)\n  cache.invoke(\"mykey\", (entry, args) -> {\n    Integer val = entry.getValue();\n\n    entry.setValue(val == null ? 1 : val + 1);\n\n    return null;\n  });",
      "language": "java",
      "name": "invoke"
    },
    {
      "code": "IgniteCache<String, Integer> cache = ignite.jcache(\"mycache\");\n\n// Increment cache value 10 times.\nfor (int i = 0; i < 10; i++)\n  cache.invoke(\"mykey\", new EntryProcessor<String, Integer, Void>() {\n    @Override \n    public Object process(MutableEntry<Integer, String> entry, Object... args) {\n      Integer val = entry.getValue();\n\n      entry.setValue(val == null ? 1 : val + 1);\n\n      return null;\n    }\n  });",
      "language": "java",
      "name": "java7 invoke"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Atomicity",
  "body": "`EntryProcessors` are executed atomically within a lock on the given cache key."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Asynchronous Support"
}
[/block]
Just like all distributed APIs in Ignite, `IgniteCache` extends [IgniteAsynchronousSupport](doc:async-support) interface and can be used in asynchronous mode.
[block:code]
{
  "codes": [
    {
      "code": "// Enable asynchronous mode.\nIgniteCache<String, Integer> asyncCache = ignite.cache(\"mycache\").withAsync();\n\n// Asynhronously store value in cache.\nasyncCache.getAndPut(\"1\", 1);\n\n// Get future for the above invocation.\nIgniteFuture<Integer> fut = asyncCache.future();\n\n// Asynchronously listen for the operation to complete.\nfut.listenAsync(f -> System.out.println(\"Previous cache value: \" + f.get()));",
      "language": "java",
      "name": "Async"
    }
  ]
}
[/block]