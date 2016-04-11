Ignite allows dependency injection of both pre-defined Ignite resources and user-defined resources. It supports field-based and method-based injection. Any resources with the proper annotations will be injected into the corresponding task, job, closure or SPI before it is initialized.
[block:api-header]
{
  "type": "basic",
  "title": "Field Based and Method Based"
}
[/block]
You can inject resources by either annotating field or method. In case you annotate the field, GridGain simply sets the value of the field at injection time (disregarding an access modifier of the field). If you annotate the method with resource annotation, it should accept an input parameter of type, corresponding to an injected resource. If it does, then the method is invoked at injection time with the appropriate resource passed as input argument.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nCollection<String> res = ignite.compute().broadcast(new IgniteCallable<String>() {\n  \t// Automatically inject ignite instance.  \n  \t@IgniteInstanceResource\n    private Ignite ignite;\n\n    @Override\n    public String call() throws Exception {\n        IgniteCache<Object, Object> cache = ignite.getOrCreateCache(CACHE_NAME);\n\n        // Do some stuff with cache.\n        ...\n    }\n});",
      "language": "java",
      "name": "Field based resource injection"
    },
    {
      "code": "",
      "language": "text",
      "name": "Method based resource injection"
    }
  ]
}
[/block]