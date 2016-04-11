Ignite allows dependency injection of both pre-defined Ignite resources and user-defined resources. It supports field-based and method-based injection. Any resources with the proper annotations will be injected into the corresponding task, job, closure or SPI before it is initialized.
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
      "name": "Field based"
    },
    {
      "code": "public class MyClusterJob implements ComputeJob {\n    ...\n    private Ignite ignite;\n    ...\n    // Inject Ignite instance.  \n    @IgniteInstanceResource\n    public void setIgnite(Ignite ignite) {\n        this.ignite = ignite;\n    }\n    ...\n}",
      "language": "java",
      "name": "Method based"
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
There are a number of pre-defined resources that you can inject:
[block:parameters]
{
  "data": {
    "h-0": "Resource",
    "h-1": "Description",
    "0-0": "`IgniteInstanceResource`",
    "0-1": "Injects current instance of Ignite API.",
    "1-0": "`CacheNameResource`",
    "1-1": "Injects grid cache name provided via `CacheConfiguration.getName()`."
  },
  "cols": 2,
  "rows": 6
}
[/block]