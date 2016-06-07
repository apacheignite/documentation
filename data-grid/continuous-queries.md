* [ContinuousQuery API](#continuousquery-api)
 * [Initial Query](#section-initial-query)
 * [Remote Filter](#section-remote-filter)
 * [Local Listener](#section-local-listener)
* [Events Delivery Guarantees](#events-delivery-guarantees)
[block:api-header]
{
  "type": "basic",
  "title": "ContinuousQuery API"
}
[/block]
Continuous queries are good for cases when you want to execute a query and then continue to get notified about the data changes that fall into your query filter.

Continuous queries are supported via `ContinuousQuery` class, which supports the following:
## Initial Query
Whenever executing continuous query, you have an option to execute initial query before starting to listen to updates. The initial query can be set via `ContinuousQuery.setInitialQuery(Query)` method and can be of any query type, [Scan](/docs/cache-queries#scan-queries), [SQL](/docs/cache-queries#sql-queries), or [TEXT](/docs/cache-queries#text-queries). This parameter is optional, and if not set, will not be used.
## Remote Filter
This filter is executed on the primary node for a given key and evaluates whether the event should be propagated to the listener. If the filter returns `true`, then the listener will be notified, otherwise the event will be skipped. Filtering events on the node on which they have occurred allows to minimize unnecessary network traffic for listener notifications. Remote filter can be set via `ContinuousQuery.setRemoteFilter(CacheEntryEventFilter<K, V>)` method.
## Local Listener
Whenever events pass the remote filter, they will be send to the client to notify the local listener there. Local listener is set via `ContinuousQuery.setLocalListener(CacheEntryUpdatedListener<K, V>)` method.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Integer, String> cache = ignite.cache(\"mycache\");\n\n// Create new continuous query.\nContinuousQuery<Integer, String> qry = new ContinuousQuery<>();\n\n// Optional initial query to select all keys greater than 10.\nqry.setInitialQuery(new ScanQuery<Integer, String>((k, v) -> k > 10)):\n\n// Callback that is called locally when update notifications are received.\nqry.setLocalListener((evts) -> \n\tevts.stream().forEach(e -> System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue())));\n\n// This filter will be evaluated remotely on all nodes.\n// Entry that pass this filter will be sent to the caller.\nqry.setRemoteFilter(e -> e.getKey() > 10);\n\n// Execute query.\ntry (QueryCursor<Cache.Entry<Integer, String>> cur = cache.query(qry)) {\n  // Iterate through existing data stored in cache.\n  for (Cache.Entry<Integer, String> e : cur)\n    System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue());\n\n  // Add a few more keys and watch a few more query notifications.\n  for (int i = 5; i < 15; i++)\n    cache.put(i, Integer.toString(i));\n}",
      "language": "java",
      "name": "continuous query"
    },
    {
      "code": "IgniteCache<Integer, String> cache = ignite.cache(CACHE_NAME);\n\n// Create new continuous query.\nContinuousQuery<Integer, String> qry = new ContinuousQuery<>();\n\nqry.setInitialQuery(new ScanQuery<Integer, String>(new IgniteBiPredicate<Integer, String>() {\n  @Override public boolean apply(Integer key, String val) {\n    return key > 10;\n  }\n}));\n\n// Callback that is called locally when update notifications are received.\nqry.setLocalListener(new CacheEntryUpdatedListener<Integer, String>() {\n  @Override public void onUpdated(Iterable<CacheEntryEvent<? extends Integer, ? extends String>> evts) {\n    for (CacheEntryEvent<Integer, String> e : evts)\n      System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue());\n  }\n});\n\n// This filter will be evaluated remotely on all nodes.\n// Entry that pass this filter will be sent to the caller.\nqry.setRemoteFilter(new CacheEntryEventFilter<Integer, String>() {\n  @Override public boolean evaluate(CacheEntryEvent<? extends Integer, ? extends String> e) {\n    return e.getKey() > 10;\n  }\n});\n\n// Execute query.\ntry (QueryCursor<Cache.Entry<Integer, String>> cur = cache.query(qry)) {\n  // Iterate through existing data.\n  for (Cache.Entry<Integer, String> e : cur)\n    System.out.println(\"key=\" + e.getKey() + \", val=\" + e.getValue());\n\n  // Add a few more keys and watch more query notifications.\n  for (int i = keyCnt; i < keyCnt + 10; i++)\n    cache.put(i, Integer.toString(i));\n}",
      "language": "java",
      "name": "java7 continuous query"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Events Delivery Guarantees"
}
[/block]
Continuous query implementation guarantees exactly once delivery of an event to the client's local listener.
It's feasible since every backup node(s) maintains an update queue in addition to the primary node. If the primary node crashes or a topology is changed for some other reason, then every backup node flushes content of its internal queue to the client making sure that there will be no event that was not delivered to the client's local listener. 

To avoid duplicate notifications, in cases when all backup nodes flush their queues to the client, Ignite manages a per-partition update counter. Once an entry in some partition is updated, counter for this partition is incremented on both primary and backups. The value of this counter is also sent along with the event notification to the client, which also maintains the copy of this mapping. If at some moment the client receives an update with the counter less than in its local map, this update is treated as a duplicate and discarded.

Once the client confirms that an event is received, the primary and backup nodes remove a record for this event from their backup queues.