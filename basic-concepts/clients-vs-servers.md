Ignite has an optional notion of **client** and **server** nodes. Server nodes participate in caching, compute execution, stream processing, etc., while the native client nodes provide ability to connect to the servers remotely. Ignite native clients allow to use the whole set of `Ignite APIs`, including near caching, transactions, compute, streaming, services, etc. from the client side.

By default, all Ignite nodes are started as `server` nodes, and `client` mode needs to be explicitly enabled.
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
Alternatively, for convenience, you can also enable or disable the client mode on the `Ignition` class itself, to allow clients and servers reuse the same configuration.
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

[block:api-header]
{
  "type": "basic",
  "title": "Managing Slow Clients"
}
[/block]
In many deployments client nodes are launched outside of the main cluster on slower machines with worse network. In these scenarios it is possible that servers will generate load (such as continuous queries notification, for example) that clients will not be able to handle, resulting in growing queue of outbound messages on servers. This may eventually cause either out-of-memory situation on server or blocking the whole cluster if back-pressure control is enabled. 

To manage these situations you can configure the maximum number of allowed outgoing messages for client nodes. If the size of outbound queue exceeds this value, such a client node will be disconnected from the cluster preventing global slowdown.

Examples below show how to configure slow client queue limit in code and XML configuration.
[block:code]
{
  "codes": [
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n\n// Configure Ignite here.\n\nTcpCommunicationSpi commSpi = new TcpCommunicationSpi();\ncommSpi.setSlowClientQueueLimit(1000);\n\ncfg.setCommunicationSpi(commSpi);",
      "language": "java"
    }
  ]
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"grid.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  <!-- Configure Ignite here. -->\n  \n  <property name=\"communicationSpi\">\n    <bean class=\"org.apache.ignite.spi.communication.tcp.TcpCommunicationSpi\">\n      <property name=\"slowClientQueueLimit\" value=\"1000\"/>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]