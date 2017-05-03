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
      "code": "K get(V val);\n\nIgniteFuture<K> getAsync(V val);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
The asynchronous operations return an instance of `IgniteFuture` or one of its subclasses. You can wait for a result of an asynchronous operation by either calling a blocking `IgniteFuture.get()` method or register a closure using `IgniteFuture.listen()` or `IgniteFuture.chain()` methods and wait while the closure gets called on the operation completion.
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
To wait for a result of an asynchronous operation in the non-blocking fashion (`IgniteFuture.get()`) register a closure using `IgniteFuture.listen()` or `IgniteFuture.chain()` methods. Once the operation gets completed the closure will be called as it's shown in the example below:
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
  "title": "Closures Execution by Thread Pools",
  "body": "If an asynchronous operation has been comp completed by the time a closure is passed to `IgniteFuture.listen()` or `IgniteFuture.chain()` methods, then the closure will be executed synchronously by the calling thread. \n\nOtherwise, the closure will be asynchronously upon the operation completion. Depending on a type of an operation,  the closure might be called by a thread from the system pool (asynchronous cache operations) or by a thread from the public pool (asynchronous Ignite Compute operations). Therefore, you should avoid calling synchronous cache and compute operations from the closure implementation. Otherwise, it might lead to a deadlock due to pools starvation."
}
[/block]