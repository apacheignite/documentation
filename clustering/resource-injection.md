Ignite allows dependency injection of both pre-defined Ignite resources and user-defined resources. It supports field-based and method-based injection. Any resources with the proper annotations will be injected into the corresponding task, job, closure or SPI before it is initialized.
[block:code]
{
  "codes": [
    {
      "code": "Collection<String> res = ignite.compute().broadcast(new IgniteCallable<String>() {\n  \t// Automatically inject ignite instance.  \n  \t@IgniteInstanceResource\n    private Ignite ignite;\n\n    @Override\n    public String call() throws Exception {\n        IgniteCache<Object, Object> cache = ignite.getOrCreateCache(CACHE_NAME);\n\n        // Do some stuff with cache.\n        ...\n    }\n});",
      "language": "java",
      "name": "Resource injection"
    }
  ]
}
[/block]