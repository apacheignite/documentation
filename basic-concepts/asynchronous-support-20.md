* [Overview](#section-overview)
* [Interfaces](#section-interfaces)
* [Listening and chaining futures](#section-listening-and-chaining-futures)
* [IgniteAsyncSupport](#section-igniteasyncsupport)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Most distributed operations on Ignite APIs can be executed either synchronously or asynchronously. Asynchronous method names end with `Async` suffix.

Asynchronous operations return instance of `IgniteFuture` or it's subclass. You may either synchronously wait for result using one of `IgniteFuture.get()` methods, or register a closure which will be executed once operation is completed using using `IgniteFuture.listen()` or `IgniteFuture.chain()` methods.
[block:code]
{
  "codes": [
    {
      "code": "K get(V val);\n\nIgniteFuture<K> getAsync(V val);",
      "language": "java",
      "name": "Example"
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "Interfaces"
}
[/block]
Asynchronous operations can be found on the following interfaces:
* `IgniteCompute`
* `IgniteCache`
* `Transaction`
* `IgniteServices`
* `IgniteMessaging`
* `IgniteEvents`
[block:api-header]
{
  "title": "Listening and chaining futures"
}
[/block]
One may register a closure which will be executed once operation is completed using using `IgniteFuture.listen()` or `IgniteFuture.chain()` methods.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCompute compute = ignite.compute();\n\n// Execute a closure asynchronously.\nIgniteFuture<String> fut = compute.callAsync(() -> {\n    return \"Hello World\";\n});\n\n// Listen for completion and print out the result.\nfut.listen(f -> System.out.println(\"Job result: \" + f.get()));",
      "language": "java",
      "name": "Example"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "Thread executing continuation",
  "body": "If future is already completed, closures passed to `listen()` and `chain()` methods will be executed synchronously in the caller thread. \n\nIf future is not completed yet, closure will be executed asynchronously in completion thread. Typically it will be a thread from system pool for cache operations, or a thread from public pool for compute operations. Therefore, you should avoid synchronous cache and compute operations in listeners for asynchronous cache and compute operations (respectively). Otherwise it may lead to a deadlock."
}
[/block]

[block:api-header]
{
  "title": "IgniteAsyncSupport"
}
[/block]
In previous versions of Apache Ignite asynchronous operations was executed using `IgniteAsyncSupport` interface. This technique is considered deprecated and should not be used anymore. `IgniteAsyncSupport` will be removed in future releases.