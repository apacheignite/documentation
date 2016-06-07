* [Overview](#overview)
* [Cache Configuration](#cache-configuration)
* [Stream Words](#stream-words)
* [Query Words](#query-words)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
In this example we will stream text into Ignite and count each individual word. We will also issue periodic SQL queries into the stream to query top 10 most popular words. 

The example will work as follows:
1. We will setup up a cache to hold the words and their counts.
2. We will setup a 5 second *sliding window* to keep the word counts only for last 5 seconds.
3. `StreamWords` program will stream text data into Ignite.
4. `QueryWords` program will query top 10 words out of the stream.
[block:api-header]
{
  "type": "basic",
  "title": "Cache Configuration"
}
[/block]
We define a `CacheConfig` class which will provide configuration to be used from both programs, `StreamWords` and `QueryWords`.  The cache will use words as keys, and counts for words as values.

Note that in this example we use a sliding window of 5 seconds for our cache. This means that words will disappear from cache after 5 seconds since they were first entered into cache.
[block:code]
{
  "codes": [
    {
      "code": "public class CacheConfig {\n  public static CacheConfiguration<String, Long> wordCache() {\n    CacheConfiguration<String, Long> cfg = new CacheConfiguration<>(\"words\");\n\n    // Index the words and their counts,\n    // so we can use them for fast SQL querying.\n    cfg.setIndexedTypes(String.class, Long.class);\n\n    // Sliding window of 5 seconds.\n    cfg.setExpiryPolicyFactory(FactoryBuilder.factoryOf(\n      new CreatedExpiryPolicy(new Duration(SECONDS, 5))));\n\n    return cfg;\n  }\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Stream Words"
}
[/block]
We define a `StreamWords` class which will be responsible to continuously read words form a local text file ("alice-in-wonderland.txt" in our case) and stream them into Ignite "words" cache.

## Streamer Configuration
1. We set `allowOverwrite` flag to `true` to make sure that existing counts can be updated.
2. We configure a `StreamTransformer` which takes currently cache count for a word and increments it by 1.
[block:code]
{
  "codes": [
    {
      "code": "public class StreamWords {\n  public static void main(String[] args) throws Exception {\n    // Mark this cluster member as client.\n    Ignition.setClientMode(true);\n\n    try (Ignite ignite = Ignition.start()) {\n      IgniteCache<String, Long> stmCache = ignite.getOrCreateCache(CacheConfig.wordCache());\n\n      // Create a streamer to stream words into the cache.\n      try (IgniteDataStreamer<String, Long> stmr = ignite.dataStreamer(stmCache.getName())) {\n        // Allow data updates.\n        stmr.allowOverwrite(true);\n\n        // Configure data transformation to count instances of the same word.\n        stmr.receiver(StreamTransformer.from((e, arg) -> {\n          // Get current count.\n          Long val = e.getValue();\n\n          // Increment current count by 1.\n          e.setValue(val == null ? 1L : val + 1);\n\n          return null;\n        }));\n\n        // Stream words from \"alice-in-wonderland\" book.\n        while (true) {\n          Path path = Paths.get(StreamWords.class.getResource(\"alice-in-wonderland.txt\").toURI());\n\n          // Read words from a text file.\n          try (Stream<String> lines = Files.lines(path)) {\n            lines.forEach(line -> {\n              Stream<String> words = Stream.of(line.split(\" \"));\n\n              // Stream words into Ignite streamer.\n              words.forEach(word -> {\n                if (!word.trim().isEmpty())\n                  stmr.addData(word, 1L);\n              });\n            });\n          }\n        }\n      }\n    }\n  }\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Query Words"
}
[/block]
We define a `QueryWords` class which will periodically query word counts form the cache.

## SQL Query
1. We use standard SQL to query the counts. 
2. Ignite SQL treats Java classes as SQL tables. Since our counts are stored as simple `Long` type, the SQL query below queries `Long` table.
3. Ignite always stores cache keys and values as `_key` and `_val` fields, so we use this syntax in our SQL query.
[block:code]
{
  "codes": [
    {
      "code": "public class QueryWords {\n  public static void main(String[] args) throws Exception {\n    // Mark this cluster member as client.\n    Ignition.setClientMode(true);\n\n    try (Ignite ignite = Ignition.start()) {\n      IgniteCache<String, Long> stmCache = ignite.getOrCreateCache(CacheConfig.wordCache());\n\n      // Select top 10 words.\n      SqlFieldsQuery top10Qry = new SqlFieldsQuery(\n        \"select _key, _val from Long order by _val desc limit 10\");\n\n      // Query top 10 popular words every 5 seconds.\n      while (true) {\n        // Execute queries.\n        List<List<?>> top10 = stmCache.query(top10Qry).getAll();\n\n        // Print top 10 words.\n        ExamplesUtils.printQueryResults(top10);\n\n        Thread.sleep(5000);\n      }\n    }\n  }\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Starting Server Nodes"
}
[/block]
In order to run the example, you need to start data nodes. In Ignite, data nodes are called `server` nodes. You can start as many server nodes as you like, but you should have at least 1 in order to run the example.

Server nodes can be started from command line as follows:
[block:code]
{
  "codes": [
    {
      "code": "bin/ignite.sh",
      "language": "shell"
    }
  ]
}
[/block]
You can also start server nodes programmatically, like so:
[block:code]
{
  "codes": [
    {
      "code": "public class ExampleNodeStartup {\n    public static void main(String[] args) throws IgniteException {\n        Ignition.start();\n    }\n}",
      "language": "java"
    }
  ]
}
[/block]