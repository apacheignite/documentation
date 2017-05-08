* [Overview](#section-overview)
* [Supported Interfaces](#section-supported-interfaces)
* [Listeners and Chaining Futures](#section-listeners-and-chaining-futures)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
A majority of Apache Ignite APIs can be used in both synchronous or asynchronous fashion. The asynchronous methods' names end with `Async` suffix.
[block:code]
{
  "codes": [
    {
      "code": "// Synchronous get\nV get(K key);\n\n// Asynchronous get\nIgniteFuture<V> getAsync(K key);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
The asynchronous operations return an instance of `IgniteFuture` or one of its subclasses. You can wait for the result of an asynchronous operation by either calling a blocking `IgniteFuture.get()` method or registering a closure using `IgniteFuture.listen()` or `IgniteFuture.chain()` methods and wait while the closure gets called on the operation completion.
[block:api-header]
{
  "title": "Supported Interfaces"
}
[/block]
The interfaces listed below can be used in synchronous or asynchronous modes:
* `IgniteCompute`
* `IgniteCache`
* `Transaction`
* `IgniteServices`
* `IgniteMessaging`
* `IgniteEvents`
[block:api-header]
{
  "title": "Listeners and Chaining Futures"
}
[/block]
To wait for the result of an asynchronous operation in a non-blocking fashion (`IgniteFuture.get()`), register a closure using `IgniteFuture.listen()` or `IgniteFuture.chain()` methods. Once the operation gets completed, the closure will be called. For example:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCompute compute = ignite.compute();\n\n// Execute a closure asynchronously.\nIgniteFuture<String> fut = compute.callAsync(() -> {\n    return \"Hello World\";\n});\n\n// Listen for completion and print out the result.\nfut.listen(f -> System.out.println(\"Job result: \" + f.get()));",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "Closures Execution and Thread Pools",
  "body": "If an asynchronous operation has been completed by the time the closure is passed to `IgniteFuture.listen()` or `IgniteFuture.chain()` method, then the closure will be executed synchronously by the calling thread. Otherwise, the closure will be executed asynchronously upon the operation completion. \n\nDepending on the type of operation,  the closure might be called by a thread from the system pool (asynchronous cache operations) or by a thread from the public pool (asynchronous Ignite Compute operations). Therefore, you should avoid calling synchronous cache and compute operations from the closure implementation. Otherwise, it may lead to a deadlock due to pools starvation.\n\nTo achieve nested execution of asynchronous Ignite Compute operations, you can take advantage of [custom thread pools](https://apacheignite.readme.io/docs/thread-pools#section-custom-thread-pools)."
}
[/block]