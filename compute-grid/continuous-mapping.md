In classical MapReduce paradigm we have a well-defined and finite set of jobs, which is known on the Map step and doesn't change throughout all computation run. But what if we have a stream of jobs instead? In this case we can still run MapReduce using Ignite's continuous mapping facility. With continuous mapping, the jobs can be generated on-the-fly when computation is already running. The newly generated jobs are processed by worker nodes as usual, and the reducer receives results just like in normal MapReduce.
[block:api-header]
{
  "type": "basic",
  "title": "Running Continuously Mapped Tasks"
}
[/block]
To use continuous mapping within your task, you need to inject `TaskContinuousMapperResource` resource into a task instance:
[block:code]
{
  "codes": [
    {
      "code": "@TaskContinuousMapperResource\nprivate TaskContinuousMapper mapper;",
      "language": "java"
    }
  ]
}
[/block]
After this, new jobs can be generated asynchronously and added to a currently running computation using `send()` methods from `TaskContinuousMapper` interface:
[block:code]
{
  "codes": [
    {
      "code": "mapper.send(new ComputeJobAdapter() {\n    @Override public Object execute() {\n        System.out.println(\"I'm a continuously-mapped job!\");\n \n        return null;\n    }\n});",
      "language": "text"
    }
  ]
}
[/block]
For continuous mapping, there are several constraints that you need to be aware of:
  *  If you initially return null from `ComputeTask.map()` method, you should send at least one job with continuous mapper before returning.
  * Continuous mapper can not be used after `ComputeTask.result()` method returned the `REDUCE` policy.
  * If `ComputeTask.result()` method returned the `WAIT` policy and all jobs are finished, then task will go to `Reduce` step and continuous mapper can not be used any more.

In other respects, the computation logic is the same as in normal MapReduce, described in [MapReduce](doc:map-reduce) chapter.