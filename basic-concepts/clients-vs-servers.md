Ignite has an optional notion of **client** and **server** nodes. Server nodes participate in caching, compute execution, stream processing, etc., while the native client nodes provide the ability to connect to the servers remotely. Ignite native clients allow to use the whole set of `Ignite APIs`, including near caching, transactions, compute, streaming, services, etc. from the client side.

By default, all Ignite nodes are started as `server` nodes, and `client` mode needs to be explicitly enabled.
[block:api-header]
{
  "type": "basic",
  "title": "Configuring Clients and Servers"
}
[/block]
You can either configure a node to be a client or a server via the `IgniteConfiguration.setClientMode(...)` property.
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
Alternatively for convenience, you may also enable or disable the client mode on the `Ignition` class itself, to allow clients and servers reuse the same configuration.
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
Whenever creating caches in Ignite, either via XML or any of the `Ignite.createCache(...)`, `Ignite.getOrCreateCache(...)` methods, Ignite will automatically deploy the distributed cache on all server nodes. 
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
By default `IgniteCompute` will execute jobs on all the server nodes. However, you may choose to execute jobs only on the server nodes or client nodes by creating a corresponding cluster group.
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
In many common deployments client nodes may be launched outside of the main cluster on slower machines with poor network. In these scenarios it is possible for servers nodes to generate load (i.e. from operations such as continuous queries notifications) that clients will not be able to handle in time hence resulting in growing queue of outbound messages on the server nodes. This may eventually cause either an out-of-memory situation on the server node or block on the whole cluster if back-pressure control is enabled. 

To handle such scenarios it is possible to configure the maximum number of allowed outgoing messages for client nodes. If the size of the outbound queue exceeds the configured value then the client node will be disconnected from the cluster to prevent global slowdown.

The examples below describe how to configure a slow client queue limit via code or XML configuration:
[block:code]
{
  "codes": [
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n\n// Configure Ignite here.\n\nTcpCommunicationSpi commSpi = new TcpCommunicationSpi();\ncommSpi.setSlowClientQueueLimit(1000);\n\ncfg.setCommunicationSpi(commSpi);",
      "language": "java"
    },
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
  "title": "Client Reconnect"
}
[/block]
Client node can disconnect from the cluster in several situations:
* in case of network problems when a client can not re-establish a connection with the server
* the connection with the server was broken for some time and the client is able to re-establish a connection with the server but server has already dropped the client node since no heartbeats were received
* slow clients can be disconnected by the server as in the example above

When a client determines that it has been disconnected from a cluster it assigns a new ID to a local node and tries to reconnect to the cluster. Note: the side effect of this is that the 'id' property of the local `ClusterNode` will change once the reconnection is successful.

While the client is in a disconnected state and a reconnection attempt is already in progress all Ignite API's will throw a special exception: `IgniteClientDisconnectedException`. This exception provides a future which will be completed when the client has successfully reconnect (`IgniteCache` API throws `CacheException` which has `IgniteClientDisconnectedException` as its cause). This future also can be obtained using the method `IgniteCluster.clientReconnectFuture()`.

Furthermore there are special events for client reconnections (these events are local, i.e. they are fired only on client nodes):
* EventType.EVT_CLIENT_NODE_DISCONNECTED
* EventType.EVT_CLIENT_NODE_RECONNECTED

The examples below show how `IgniteClientDisconnectedException` can be used:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCompute compute = ignite.compute();\n\nwhile (true) {\n    try {\n        compute.run(job);\n    }\n    catch (IgniteClientDisconnectedException e) {\n        e.reconnectFuture().get(); // Wait for reconnect.\n\n        // Can proceed and use the same IgniteCompute instance.\n    }\n}\n",
      "language": "java",
      "name": "Compute"
    },
    {
      "code": "IgniteCache cache = ignite.getOrCreateCache(new CacheConfiguration<>());\n\nwhile (true) {\n  try {\n    cache.put(key, val);\n  }\n  catch (CacheException e) {\n    if (e.getCause() instanceof IgniteClientDisconnectedException) {\n      IgniteClientDisconnectedException cause =\n        (IgniteClientDisconnectedException)e.getCause();\n\n      cause.reconnectFuture().get(); // Wait for reconnect.\n\n      // Can proceed and use the same IgniteCache instance.\n    }\n  }\n}\n",
      "language": "java",
      "name": "Cache"
    }
  ]
}
[/block]
Automatic client reconnection can be disabled using 'clientReconnectDisabled' property on `TcpDiscoverySpi`, if reconnection is disabled then the client node is stopped when it determines that it has disconnected from the cluster.

The example below shows how to disable client reconnection:
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
  "title": "Forcing Server Mode On Client Nodes"
}
[/block]
Client nodes will require alive server nodes in a topology in order to start.

It is possible to start client nodes regardless of server nodes being present by forcing server mode discovery on client nodes as in the example below:
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
In this case discovery will happen as if all nodes in topology were server nodes.
[block:callout]
{
  "type": "warning",
  "title": "Important Notice",
  "body": "In such a case all addresses configured for discovery would need be mutually reachable in order for the discovery to work properly."
}
[/block]