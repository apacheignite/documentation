* [Overview](#overview)
* [IgniteAsyncSupport](#igniteasyncsupport)
 * [Compute Grid Example](#section-compute-grid-example)
 * [Data Grid Example](#section-data-grid-example)
* [@IgniteAsyncSupported Annotation](#igniteasyncsupported)
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

[block:callout]
{
  "type": "warning",
  "title": "Method Return Values",
  "body": "Note, that if async mode is enabled, actual synchronously returned values of methods should be ignored. The only way to obtain a return value from an asynchronous operation is from the `future()` method."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "IgniteCompute compute = ignite.compute();\n\n// Execute a job and wait for the result.\nString res = compute.call(() -> {\n  // Print hello world on some cluster node.\n\tSystem.out.println(\"Hello World\");\n  \n  return \"Hello World\";\n});",
      "language": "java",
      "name": "Synchronous"
    }
  ]
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "// Enable asynchronous mode.\nIgniteCompute asyncCompute = ignite.compute().withAsync();\n\n// Asynchronously execute a job.\nasyncCompute.call(() -> {\n  // Print hello world on some cluster node and wait for completion.\n\tSystem.out.println(\"Hello World\");\n  \n  return \"Hello World\";\n});\n\n// Get the future for the above invocation.\nIgniteFuture<String> fut = asyncCompute.future();\n\n// Asynchronously listen for completion and print out the result.\nfut.listen(f -> System.out.println(\"Job result: \" + f.get()));",
      "language": "java",
      "name": "Asynchronous"
    }
  ]
}
[/block]
## Data Grid Example
Here is the data grid example for synchronous and asynchronous invocations.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<String, Integer> cache = ignite.cache(\"mycache\");\n\n// Synchronously store value in cache and get previous value.\nInteger val = cache.getAndPut(\"1\", 1);",
      "language": "java",
      "name": "Synchronous"
    }
  ]
}
[/block]
Here is how you would make the above invocation asynchronous.
[block:code]
{
  "codes": [
    {
      "code": "// Enable asynchronous mode.\nIgniteCache<String, Integer> asyncCache = ignite.cache(\"mycache\").withAsync();\n\n// Asynchronously store value in cache.\nasyncCache.getAndPut(\"1\", 1);\n\n// Get future for the above invocation.\nIgniteFuture<Integer> fut = asyncCache.future();\n\n// Asynchronously listen for the operation to complete.\nfut.listen(f -> System.out.println(\"Previous cache value: \" + f.get()));",
      "language": "java",
      "name": "Asynchronous"
    }
  ]
}
[/block]