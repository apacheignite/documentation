* [Overview](#overview)
* [IgniteDataStreamer](##ignitedatastreamer)
 * [Allow Overwrite](#section-allow-overwrite)
* [StreamReceiver](#streamreceiver)
* [StreamTransformer](#streamtransformer)
* [StreamVisitor](#streamvisitor)
* [Integrations With Existed Streaming Technologies](#integrations-with-existed-streaming-technologies)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Data streamers are defined by `IgniteDataStreamer` API and are built to inject large amounts of continuous streams of data into Ignite caches. Data streamers are built in a scalable and fault-tolerant fashion and provide **at-least-once-guarantee** semantics for all the data streamed into Ignite.
[block:api-header]
{
  "type": "basic",
  "title": "IgniteDataStreamer"
}
[/block]
The main abstraction for fast streaming of large amounts of data into Ignite is `IgniteDataStreamer`, which internally will properly batch keys together and collocate those batches with nodes on which the data will be cached. 

The high loading speed is achieved with the following techniques:
  * Entries that are mapped to the same cluster member will be batched together in a buffer.
  * Multiple buffers can coexist at the same time.
  * To avoid running out of memory, data streamer has a maximum number of buffers it can process concurrently.

To add data to the data streamer, you should call `IgniteDataStreamer.addData(...)` method.
[block:code]
{
  "codes": [
    {
      "code": "// Get the data streamer reference and stream data.\ntry (IgniteDataStreamer<Integer, String> stmr = ignite.dataStreamer(\"myStreamCache\")) {    \n    // Stream entries.\n    for (int i = 0; i < 100000; i++)\n        stmr.addData(i, Integer.toString(i));\n}",
      "language": "java"
    }
  ]
}
[/block]
## Allow Overwrite
By default, the data streamer will not overwrite existing data, which means that if it will encounter an entry that is already in cache, it will skip it. This is the most efficient and performant mode, as the data streamer does not have to worry about data versioning in the background.

If you anticipate that the data may already be in the streaming cache and you need to overwrite it, you should set `IgniteDataStreamer.allowOverwrite(true)` parameter.
[block:api-header]
{
  "type": "basic",
  "title": "StreamReceiver"
}
[/block]
For cases when you need to execute some custom logic instead of just adding new data, you can take advantage of `StreamReceiver` API. 

Stream receivers allow you to react to the streamed data in collocated fashion, directly on the nodes where it will be cached. You can change the data or add any custom pre-processing logic to it, before putting the data into cache.
[block:callout]
{
  "type": "info",
  "body": "Note that `StreamReceiver` does not put data into cache automatically. You need to call any of the `cache.put(...)` methods explicitly."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "StreamTransformer"
}
[/block]
`StreamTransformer` is a convenience implementation of `StreamReceiver` which updates data in the stream cache based on its previous value. The update is collocated, i.e. it happens exactly on the cluster node where the data is stored.

In the example below, we use `StreamTransformer` to increment a counter for each distinct word found in the text stream.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration cfg = new CacheConfiguration(\"wordCountCache\");\n\nIgniteCache<String, Long> stmCache = ignite.getOrCreateCache(cfg);\n\ntry (IgniteDataStreamer<String, Long> stmr = ignite.dataStreamer(stmCache.getName())) {\n  // Allow data updates.\n  stmr.allowOverwrite(true);\n\n  // Configure data transformation to count instances of the same word.\n  stmr.receiver(StreamTransformer.from((e, arg) -> {\n    // Get current count.\n    Long val = e.getValue();\n\n    // Increment count by 1.\n    e.setValue(val == null ? 1L : val + 1);\n\n    return null;\n  }));\n\n  // Stream words into the streamer cache.\n  for (String word : text)\n    stmr.addData(word, 1L);\n}",
      "language": "java",
      "name": "transformer"
    },
    {
      "code": "CacheConfiguration cfg = new CacheConfiguration(\"wordCountCache\");\n\nIgniteCache<Integer, Long> stmCache = ignite.getOrCreateCache(cfg);\n\ntry (IgniteDataStreamer<String, Long> stmr = ignite.dataStreamer(stmCache.getName())) {\n  // Allow data updates.\n  stmr.allowOverwrite(true);\n\n  // Configure data transformation to count instances of the same word.\n  stmr.receiver(new StreamTransformer<String, Long>() {\n    @Override public Object process(MutableEntry<String, Long> e, Object... args) {\n      // Get current count.\n      Long val = e.getValue();\n\n      // Increment count by 1.\n      e.setValue(val == null ? 1L : val + 1);\n\n      return null;\n    }\n  });\n\n  // Stream words into the streamer cache.\n  for (String word : text)\n    stmr.addData(word, 1L);",
      "language": "java",
      "name": "java7 transformer"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "StreamVisitor"
}
[/block]
`StreamVisitor` is also a convenience implementation of `StreamReceiver` which visits every key-value tuple in the stream. Note, that the visitor does not update the cache. If the tuple needs to be stored in the cache, then any of the `cache.put(...)` methods should be called explicitly.

In the example below, we have 2 caches: "marketData", and "instruments". We receive market data ticks and put them into the streamer for the "marketData" cache. The `StreamVisitor` for the "marketData" streamer is invoked on the cluster member mapped to the particular market symbol. Upon receiving individual market ticks it updates the "instrument" cache with latest market price.

Note, that we do not update "marketData" cache at all, leaving it empty. We simply use for collocated processing of the market data within the cluster directly on the node where the data will be stored.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<String, Double> mrktDataCfg = new CacheConfiguration<>(\"marketData\");\nCacheConfiguration<String, Double> instCfg = new CacheConfiguration<>(\"instruments\");\n\n// Cache for market data ticks streamed into the system.\nIgniteCache<String, Double> mrktData = ignite.getOrCreateCache(mrktDataCfg);\n\n// Cache for financial instruments.\nIgniteCache<String, Double> insts = ignite.getOrCreateCache(instCfg);\n\ntry (IgniteDataStreamer<String, Integer> mktStmr = ignite.dataStreamer(\"marketData\")) {\n  // Note that we do not populate 'marketData' cache (it remains empty).\n  // Instead we update the 'instruments' cache based on the latest market price.\n  mktStmr.receiver(StreamVisitor.from((cache, e) -> {\n    String symbol = e.getKey();\n    Double tick = e.getValue();\n\n    Instrument inst = instCache.get(symbol);\n\n    if (inst == null)\n      inst = new Instrument(symbol);\n\n    // Update instrument price based on the latest market tick.\n    inst.setHigh(Math.max(inst.getLatest(), tick);\n    inst.setLow(Math.min(inst.getLatest(), tick);\n    inst.setLatest(tick);\n\n    // Update instrument cache.\n    instCache.put(symbol, inst);\n  }));\n\n  // Stream market data into Ignite.\n  for (Map.Entry<String, Double> tick : marketData)\n      mktStmr.addData(tick);\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Integrations With Existed Streaming Technologies"
}
[/block]
Apache Ignite has a variety of integrations with well-known streaming products and technologies like Kafka, Camel or JMS which enables to inject streams of data into Ignite easily and efficiently.

* [Kafka Streamer](https://apacheignite-mix.readme.io/docs/kafka-streamer)
* [Camel Streamer](https://apacheignite-mix.readme.io/docs/camel-streamer) 
* [JMS Streamer](https://apacheignite-mix.readme.io/docs/jms-data-streamer) 
* [MQTT Streamer]https://apacheignite-mix.readme.io/docs/mqtt-streamer) 
* [Storm Streamer](https://apacheignite-mix.readme.io/docs/storm-streamer) 
* [Flink Streamer](https://apacheignite-mix.readme.io/docs/flink-streamer) 
* [Twitter Streamer](https://apacheignite-mix.readme.io/docs/twitter-streamer) 
* [Flume Sink](https://apacheignite-mix.readme.io/docs/flume-sink)