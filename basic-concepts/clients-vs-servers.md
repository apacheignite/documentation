* [Overview](#overview)
* [Configuring Clients and Servers](#configuring-clients-and-servers)
* [Creating Distributed Caches](#creating-distributed-caches)
* [Computing on Clients or Servers](#computing-on-clients-or-servers)
* [Managing Slow Clients](#managing-slow-clients)
* [Client Reconnection](#client-reconnection)
* [Forcing Server Mode on Client Nodes](#forcing-server-mode-on-client-nodes)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite has an optional notion of **client** and **server** nodes. Server nodes participate in caching, compute execution, stream processing, etc., while the native client nodes provide ability to connect to the servers remotely. Ignite native clients allow using the whole set of `Ignite APIs`, including near caching, transactions, compute, streaming, services, etc. from the client side.

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
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...   \n    <!-- Enable client mode. -->\n    <property name=\"clientMode\" value=\"true\"/>\n    ...\n</bean>",
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
By default `IgniteCompute` will execute jobs on all the server nodes. However, you can choose to execute jobs only on server nodes or only on client nodes by creating a corresponding cluster group.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCompute compute = ignite.compute();\n\n// Execute computation on the server nodes (default behavior).\ncompute.broadcast(() -> System.out.println(\"Hello Server\"));",
      "language": "java",
      "name": "Compute on Servers"
    },
    {
      "code": "ClusterGroup clientGroup = ignite.cluster().forClients();\n\nIgniteCompute clientCompute = ignite.compute(clientGroup);\n\n// Execute computation on the client nodes.\nclientCompute.broadcast(() -> System.out.println(\"Hello Client\"));",
      "language": "java",
      "name": "Compute on Clients"
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

To manage these situations, you can configure the maximum number of allowed outgoing messages for client nodes. If the size of outbound queue exceeds this value, such a client node will be disconnected from the cluster preventing global slowdown.

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

[block:api-header]
{
  "type": "basic",
  "title": "Client Reconnection"
}
[/block]
Client nodes can get disconnected from the cluster in several cases:
*  When a client node cannot re-establish the connection with the server node due to network issues.
* Connection with the server node was broken for some time; the client node is able to re-establish the connection with the server, but server already dropped the client node since the server did not receive client heartbeats
* Slow clients can be kicked out by server nodes.

When a client determines that it is disconnected from the cluster, it assigns a new node 'id' to itself and tries to reconnect to the cluster. Note that this has side effect - the 'id' property of the local `ClusterNode` will change in case of client reconnection. This means that any application logic that relied on the 'id' value may be affected.

While a client is in a disconnected state and an attempt to reconnect is in progress, the Ignite API throws  a special exception - `IgniteClientDisconnectedException`. This exception provides `future` which will be completed when the client reconnects with the cluster (`IgniteCache` API throws `CacheException` which has `IgniteClientDisconnectedException` as its cause). This `future` can also be obtained using the `IgniteCluster.clientReconnectFuture()` method.

Also, there are special events for client reconnection (these events are local, i.e. they are fired only on the client node):
* EventType.EVT_CLIENT_NODE_DISCONNECTED
* EventType.EVT_CLIENT_NODE_RECONNECTED

The following example shows how to use `IgniteClientDisconnectedException`.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCompute compute = ignite.compute();\n\nwhile (true) {\n    try {\n        compute.run(job);\n    }\n    catch (IgniteClientDisconnectedException e) {\n        e.reconnectFuture().get(); // Wait for reconnection.\n\n        // Can proceed and use the same IgniteCompute instance.\n    }\n}\n",
      "language": "java",
      "name": "Compute"
    },
    {
      "code": "IgniteCache cache = ignite.getOrCreateCache(new CacheConfiguration<>());\n\nwhile (true) {\n  try {\n    cache.put(key, val);\n  }\n  catch (CacheException e) {\n    if (e.getCause() instanceof IgniteClientDisconnectedException) {\n      IgniteClientDisconnectedException cause =\n        (IgniteClientDisconnectedException)e.getCause();\n\n      cause.reconnectFuture().get(); // Wait for reconnection.\n\n      // Can proceed and use the same IgniteCache instance.\n    }\n  }\n}\n",
      "language": "java",
      "name": "Cache"
    }
  ]
}
[/block]
Automatic client reconnection can be disabled using the 'clientReconnectDisabled' property on `TcpDiscoverySpi`. When reconnection is disabled, client node is stopped.
[block:code]
{
  "codes": [
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n\n// Configure Ignite here.\n\nTcpDiscoverySpi discoverySpi = new TcpDiscoverySpi();\n\ndiscoverySpi.setClientReconnectDisabled(true);\n\ncfg.setDiscoverySpi(discoverySpi);\n",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Forcing Server Mode on Client Nodes"
}
[/block]
Client nodes require live server nodes in the topology to start.

However, to start a client node without a running server node,  you can force server mode discovery on client nodes the following way:
[block:code]
{
  "codes": [
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setClientMode(true);\n\n// Configure Ignite here.\n\nTcpDiscoverySpi discoverySpi = new TcpDiscoverySpi();\n\ndiscoverySpi.setForceServerMode(true);\n\ncfg.setDiscoverySpi(discoverySpi);",
      "language": "java"
    }
  ]
}
[/block]
In this case discovery will happen as if all the nodes in topology were server nodes.
[block:callout]
{
  "type": "warning",
  "title": "",
  "body": "In this case, all addresses used by the discovery SPI on all nodes should be mutually reachable in order for the discovery to work properly."
}
[/block]