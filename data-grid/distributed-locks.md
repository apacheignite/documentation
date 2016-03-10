Cache transactions will acquire locks implicitly. However, there are cases when explicit locks are more useful. The `lock()` method of `IgniteCache` API returns an instance of `java.util.concurrent.locks.Lock`, that lets you define explicit distributed locks for any given key. Locks can also be acquired on a collection of objects using `IgniteCache.lockAll()` method.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<String, Integer> cache = ignite.cache(\"myCache\");\n\n// Create a lock for the given key.\nLock lock = cache.lock(\"keyLock\");\ntry {\n  \t// Aquire the lock.\n    lock.lock();\n  \n    cache.put(\"Hello\", 11);\n    cache.put(\"World\", 22);\n}\nfinally {\n  \t// Release the lock.\n    lock.unlock();\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Atomicity Mode",
  "body": "In Ignite, locks are supported only for `TRANSACTIONAL` atomicity mode, which can be configured via `atomicityMode` property of `CacheConfiguration`."
}
[/block]
## Locks and Transactions
Explicit locks are not transactional and cannot not be used from within transactions (exception will be thrown). If you do need explicit locking within transactions, then you should use `TransactionConcurrency.PESSIMISTIC` concurrency control for transactions which will acquire explicit locks for relevant cache operations.