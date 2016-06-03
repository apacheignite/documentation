- [Field Based and Method Based](doc:resource-injection#field-based-and-method-based)
- [Pre-defined Resources](doc:resource-injection#pre-defined-resources)

Ignite allows dependency injection of pre-defined Ignite resources, and supports field-based as well as method-based injection. Any resources with the proper annotations will be injected into the corresponding task, job, closure or SPI before it is initialized.
[block:api-header]
{
  "type": "basic",
  "title": "Field Based and Method Based"
}
[/block]
You can inject resources by either annotating a field or a method. In case you annotate a field, Ignite simply sets the value of the field at injection time (disregarding an access modifier of the field). If you annotate a method with resource annotation, it should accept an input parameter of type corresponding to an injected resource. If it does, then the method is invoked at injection time with the appropriate resource passed as input argument.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nCollection<String> res = ignite.compute().broadcast(new IgniteCallable<String>() {\n  \t// Inject Ignite instance.  \n  \t@IgniteInstanceResource\n    private Ignite ignite;\n\n    @Override\n    public String call() throws Exception {\n        IgniteCache<Object, Object> cache = ignite.getOrCreateCache(CACHE_NAME);\n\n        // Do some stuff with cache.\n        ...\n    }\n});",
      "language": "java",
      "name": "Field Based"
    },
    {
      "code": "public class MyClusterJob implements ComputeJob {\n    ...\n    private Ignite ignite;\n    ...\n    // Inject Ignite instance.  \n    @IgniteInstanceResource\n    public void setIgnite(Ignite ignite) {\n        this.ignite = ignite;\n    }\n    ...\n}",
      "language": "java",
      "name": "Method Based"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Pre-defined Resources"
}
[/block]
There are a number of pre-defined Ignite resources that you can inject:
[block:parameters]
{
  "data": {
    "0-0": "`CacheNameResource`",
    "h-0": "Resource",
    "h-1": "Description",
    "0-1": "Injects grid cache name provided via `CacheConfiguration.getName()`.",
    "1-0": "`CacheStoreSessionResource`",
    "1-1": "Injects current `CacheStoreSession` instance.",
    "2-0": "`IgniteInstanceResource`",
    "2-1": "Injects current instance of Ignite API.",
    "3-0": "`JobContextResource`",
    "3-1": "Injects instance of `ComputeJobContext`. Job context holds useful information about a particular job execution. For example, you can get the name of the cache containing the entry for which a job was [Collocated](doc:collocate-compute-and-data).",
    "4-0": "`LoadBalancerResource`",
    "4-1": "Injects instance of `IgniteLogger` which is used to write messages to a local node's log.",
    "5-0": "`ServiceResource`",
    "5-1": "Injects Ignite service by specified service name.",
    "6-0": "`SpringApplicationContextResource`",
    "6-1": "Injects Spring's `ApplicationContext` resource.",
    "7-0": "`SpringResource`",
    "7-1": "Injects resource from Spring's `ApplicationContext`. Use it whenever you would like to access a bean specified in Spring's application context  XML configuration.",
    "8-0": "`TaskContinuousMapperResource`",
    "8-1": "Injects an instance of `ComputeTaskContinuousMapper`. Continuous mapping allows to emit jobs from the task at any point, even after initial *map* phase.",
    "9-0": "`TaskSessionResource`",
    "9-1": "Injects instance of `ComputeTaskSession` resource which defines a distributed session for a particular task execution."
  },
  "cols": 2,
  "rows": 10
}
[/block]