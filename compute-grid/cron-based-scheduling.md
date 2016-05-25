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
  "body": "In current implementation, Cron shorthands (@hourly, @daily, @weekly...) are not supported, and a minimal scheduling time unit is 1 minute.",
  "title": "Cron shorthands"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "SchedulerFuture"
}
[/block]
The `IgniteScheduler.scheduleLocal()` methods return `SchedulerFuture`, which has a bunch of useful methods for monitoring scheduled execution and getting results. To cancel periodic execution, you also use the future.
[block:code]
{
  "codes": [
    {
      "code": "SchedulerFuture<?> fut = ignite.scheduler().scheduleLocal(new Runnable() {\n    @Override public void run() {\n        ...\n    }\n}, \"0 0 * * *\");\n\nSystem.out.println(\"The task will be next executed on \" + new Date(fut.nextExecutionTime()));\n\nfut.get(); // Wait for next execution to finish.\n\nfut.cancel(); // Cancel periodic execution.",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Syntax Extension"
}
[/block]
Ignite introduces an extension to Cron syntax, which allows to specify an initial delay in seconds and a number of runs. These two optional numbers go in curly braces, comma-separated, before the Cron specification. An example below specifies execution 5 times each minute with an initial 2 seconds delay.
[block:code]
{
  "codes": [
    {
      "code": "{2, 5} * * * * *",
      "language": "text"
    }
  ]
}
[/block]