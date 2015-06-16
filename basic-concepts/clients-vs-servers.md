Ignite has an optional notion of **client** and **server** nodes. By default, all Ignite nodes are started as *server* nodes, which means that they will automatically participate in distributed caches. 
[block:api-header]
{
  "type": "basic",
  "title": "Configuring Clients and Servers"
}
[/block]
You can configure a node to be either a client or a server via `IgniteConfiguration.setClientMode(...)` property.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...   \n    <!-- Enable client mdoe. -->\n    <property name=\"clientMode\" value=\"true\"/>\n    ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n\n// Enable client mode.\ncfg.setClientMode(true);\n\n// Start Ignite in client mode.\nIgnite ignite = Ignition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
Alternatively, for convenience, you can also enable client mode on the `Ignition` class itself, to allow clients and servers reuse the same configuration.
[block:code]
{
  "codes": [
    {
      "code": "Ignition.setClientMode(true);\n\n// Start Ignite in client mode.\nIgnite ignite = Ignition.start();",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Creating Distributed Caches"
}
[/block]
Whenever creating caches in Ignite, either in XML or via any of the `Ignite.createCache(...)` or `Ignite.getOrCreateCache(...)` methods, Ignite will automatically deploy the distributed cache on all server nodes. 
[block:callout]
{
  "type": "success",
  "body": "Once a distributed cache is created, it will be automatically deployed on all the existing and future **server** nodes."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "// Enable client mode locally.\nIgnition.setClientMode(true);\n\n// Start Ignite in client mode.\nIgnite ignite = Ignition.start();\n\nCacheConfiguration cfg = new CacheConfiguration(\"myCache\");\n\n// Set required cache configuration properties.\n...\n\n// Create cache on all the existing and future server nodes.\n// Note that since the local node is a client, it will not \n// be caching any data.\nIgniteCache<?, ?> cache = ignite.getOrCreateCache(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Computing on Clients or Servers"
}
[/block]
By default `IgniteCompute` will execute jobs on all the cluster nodes. However, you can choose to execute jobs only on server nodes by creating a cluster group for the servers. You can also create a cluster group for all the client nodes as well.
[block:code]
{
  "codes": [
    {
      "code": "ClusterGroup serverGroup = ignite.cluster().forServers();\nClusterGroup clientGroup = ignite.cluster().forClients();\n\nIgniteCompute serverCompute = ignite.compute(serverGroup);\nIgniteCompute clientCompute = ignite.compute(clientGroup);\n\n// Execute computation on the server nodes.\nserverCompute.broadcast(() -> System.out.println(\"Hello Server\"));\n\n// Execute computation on the client nodes.\nclientCompute.broadcast(() -> System.out.println(\"Hello Client\"));",
      "language": "java"
    }
  ]
}
[/block]