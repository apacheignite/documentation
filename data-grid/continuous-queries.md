* [Continuous Queries](#section-continuous-queries)
 * [Initial Query](#section-initial-query)
 * [Remote Filter](#section-remote-filter)
 * [Local Listener](#section-local-listener)
* [Events Delivery Guarantees](#events-delivery-guarantees)
* [Example](#example)
[block:api-header]
{
  "type": "basic",
  "title": "Continuous Queries"
}
[/block]
Continuous queries enable you to listen to the data modifications happened on Ignite's distributed caches side. Once a continuous query is started you'll get notified of all the modifications happened to the data set according to the query filter if any.

Continuous queries functionality is available via `ContinuousQuery` class that is elaborated below.

## Initial Query

Whenever a continuous query is prepared for the execution, you have an option to specify an initial query that will be executed before the continuous query will be registered in the cluster and before you will receive the updates.

The initial query can be set with `ContinuousQuery.setInitialQuery(Query)` method and can be of any query type: [Scan](/docs/cache-queries#section-scan-queries), [SQL](doc:sql-grid) , or [TEXT](/docs/cache-queries#text-queries).

## Remote Filter

This filter is executed on primary and backup nodes for a given key and evaluates whether an update should be propagated as an event to query's local listener.

If the filter returns `true`, then the local listener will be notified. Otherwise, the notification will be skipped. Updates filtering on specific primary and backup nodes, on which they occur, allows to reduce unnecessary network traffic for between primary/backup nodes and local listeners executed on an application side.

Remote filter can be set via `ContinuousQuery.setRemoteFilter(CacheEntryEventFilter<K, V>)` method.

## Local Listener

When a cache gets modified (an entry is inserted, updated or deleted), an event related to the update will be sent the continuous query's local listener so that your application can react accordingly.

Whenever events pass the remote filter, they will be sent to the client to notify the local listener there.
The local listener is set via `ContinuousQuery.setLocalListener(CacheEntryUpdatedListener<K, V>)` method.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Integer, String> cache = ignite.cache(\"mycache\");\n\n// Creating a continuous query.\nContinuousQuery<Integer, String> qry = new ContinuousQuery<>();\n\n// Setting an optional initial query. \n// The query will return entries for the keys greater than 10.\nqry.setInitialQuery(new ScanQuery<Integer, String>((k, v) -> k > 10)):\n\n// Local listener that is called locally when an update notification is received.\nqry.setLocalListener((evts) -> \n\tevts.stream().forEach(e -> System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue())));\n\n// This filter will be evaluated remotely on all nodes.\n// Entry that pass this filter will be sent to the local listener.\nqry.setRemoteFilter(e -> e.getKey() > 10);\n\n// Executing the query.\ntry (QueryCursor<Cache.Entry<Integer, String>> cur = cache.query(qry)) {\n  // Iterating over existing data stored in cache.\n  for (Cache.Entry<Integer, String> e : cur)\n    System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue());\n\n  // Adding a few more cache entries.\n  // As a result, the local listener above will be called.\n  for (int i = 5; i < 15; i++)\n    cache.put(i, Integer.toString(i));\n}",
      "language": "java",
      "name": "continuous query"
    },
    {
      "code": "IgniteCache<Integer, String> cache = ignite.cache(CACHE_NAME);\n\n// // Creating a continuous query.\nContinuousQuery<Integer, String> qry = new ContinuousQuery<>();\n\n// Setting an optional initial query. \n// The query will return entries for the keys greater than 10.\nqry.setInitialQuery(new ScanQuery<Integer, String>(\n  new IgniteBiPredicate<Integer, String>() {\n  @Override public boolean apply(Integer key, String val) {\n    return key > 10;\n  }\n}));\n\n// Local listener that is called locally when an update notification is received.\nqry.setLocalListener(new CacheEntryUpdatedListener<Integer, String>() {\n  @Override public void onUpdated(Iterable<CacheEntryEvent<? extends Integer, ? extends String>> evts) {\n    for (CacheEntryEvent<Integer, String> e : evts)\n      System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue());\n  }\n});\n\n// This filter will be evaluated remotely on all nodes.\n// Entry that pass this filter will be sent to the local listener.\nqry.setRemoteFilter(new CacheEntryEventFilter<Integer, String>() {\n  @Override public boolean evaluate(CacheEntryEvent<? extends Integer, ? extends String> e) {\n    return e.getKey() > 10;\n  }\n});\n\n// Execute query.\ntry (QueryCursor<Cache.Entry<Integer, String>> cur = cache.query(qry)) {\n  // Iterating over existing data stored in cache.\n  for (Cache.Entry<Integer, String> e : cur)\n    System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue());\n\n  // Adding a few more cache entries.\n  // As a result, the local listener above will be called.\n  for (int i = keyCnt; i < keyCnt + 10; i++)\n    cache.put(i, Integer.toString(i));\n}",
      "language": "java",
      "name": "java7 continuous query"
    }
  ]
}
[/block]
## Events Delivery Guarantees

Continuous query implementation guarantees exactly once delivery of an event to the client's local listener.

It's feasible since every backup node(s) maintains an update queue in addition to the primary node. If the primary node crashes or a topology is changed for some other reason, then every backup node flushes content of its internal queue to the client making sure that there will be no event that was not delivered to the client's local listener. 

To avoid duplicate notifications, in cases when all backup nodes flush their queues to the client, Ignite manages a per-partition update counter. Once an entry in some partition is updated, a counter for this partition is incremented on both primary and backups. The value of this counter is also sent along with the event notification to the client, which also maintains the copy of this mapping. If at some moment the client receives an update with the counter less than in its local map, this update is treated as a duplicate and discarded.

Once the client confirms that an event is received, the primary and backup nodes remove a record for this event from their backup queues.
[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
A complete example that demonstrates the usage of continuous queries, covered under this documentation section, is delivered as a part of every Apache Ignite distribution and named `CacheContinuousQueryExample`. The example is [available](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/datagrid/CacheContinuousQueryExample.java) in Git Hub as well.