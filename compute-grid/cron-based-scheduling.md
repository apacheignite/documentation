Instances of `Runnable` and `Callable` can be scheduled for periodic execution on local node using `IgniteScheduler.scheduleLocal()` methods and `Cron` syntax. An example below triggers periodic cache metrics reporting on all available nodes: 
[block:code]
{
  "codes": [
    {
      "code": "ignite.compute().broadcast(new IgniteCallable<Object>() {\n    @IgniteInstanceResource\n    Ignite ignite;\n\n    @Override\n    public Object call() throws Exception {\n        ignite.scheduler().scheduleLocal(new Runnable() {\n            @Override public void run() {\n                sendReportWithCacheMetrics(cache.metrics());\n            }\n        }, \"0 0   *\");\n        return null;\n    }\n});",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "In current implementation, Cron shorthands (@hourly, @daily, @weekly...) are not supported, and a minimal scheduling time unit is 1 minute."
}
[/block]