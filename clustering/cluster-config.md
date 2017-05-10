* [Overview](#overview)
* [Multicast Based Discovery](doc:cluster-config#multicast-based-discovery)
* [Static IP Based Discovery](doc:cluster-config#static-ip-based-discovery)
* [Multicast and Static IP Based Discovery](doc:cluster-config#multicast-and-static-ip-based-discovery)
* [Isolated Ignite Clusters on Same Set of Machines](doc:cluster-config#isolated-ignite-clusters-on-same-set-of-machin)
* [Apache jclouds Based Discovery](doc:cluster-config#apache-jclouds-based-discovery)
* [Amazon S3 Based Discovery](doc:cluster-config#amazon-s3-based-discovery)
* [Google Cloud Storage Based Discovery](doc:cluster-config#google-cloud-storage-based-discovery)
* [JDBC Based Discovery](doc:cluster-config#jdbc-based-discovery)
* [Shared File System Based Discovery](doc:cluster-config#shared-file-system-based-discovery)
* [Kubernetes Service Based Discovery](#kubernetes-service-based-discovery)
* [ZooKeeper Based Discovery](doc:cluster-config#zookeeper-based-discovery)
* [Failure Detection Timeout](doc:cluster-config#failure-detection-timeout)
* [Configuration](doc:cluster-config#configuration)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
In Ignite, nodes can discover each other by using `DiscoverySpi`. Ignite provides `TcpDiscoverySpi` as a default implementation of `DiscoverySpi` that uses TCP/IP for node discovery. Discovery SPI can be configured for Multicast and Static IP based node discovery.
[block:api-header]
{
  "type": "basic",
  "title": "Multicast Based Discovery"
}
[/block]
`TcpDiscoveryMulticastIpFinder` uses Multicast to discover other nodes in the grid and is the default IP finder. You should not have to specify it unless you plan to override default settings. Here is an example of how to configure this finder via Spring XML file or programmatically from Java:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"discoverySpi\">\n    <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n      <property name=\"ipFinder\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.multicast.TcpDiscoveryMulticastIpFinder\">\n          <property name=\"multicastGroup\" value=\"228.10.10.157\"/>\n        </bean>\n      </property>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "TcpDiscoverySpi spi = new TcpDiscoverySpi();\n \nTcpDiscoveryVmIpFinder ipFinder = new TcpDiscoveryMulticastIpFinder();\n \nipFinder.setMulticastGroup(\"228.10.10.157\");\n \nspi.setIpFinder(ipFinder);\n \nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default discovery SPI.\ncfg.setDiscoverySpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Static IP Based Discovery"
}
[/block]
For cases when Multicast is disabled, `TcpDiscoveryVmIpFinder` should be used with pre-configured list of IP addresses.

You are only required to provide at least one IP address of a remote node, but usually it is advisable to provide 2 or 3 addresses of grid nodes that you plan to start at some point of time in the future. Once a connection to any of the provided IP addresses is established, Ignite will automatically discover all other grid nodes.

[block:callout]
{
  "type": "warning",
  "body": "By default the `TcpDiscoveryVmIpFinder` is used in `non-shared` mode. If you plan to start a server node then in this mode the list of IP addresses should contain an address of the local node as well. It will let the node not to wait while other nodes join the cluster but rather become the first cluster node and operate normally."
}
[/block]
Here is an example of how to configure this finder via Spring XML file or programmatically from Java:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"discoverySpi\">\n    <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n      <property name=\"ipFinder\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.vm.TcpDiscoveryVmIpFinder\">\n          <property name=\"addresses\">\n            <list>\n              <!--\n \t\t\t\t\t\t\t\t\tExplicitly specifying address of a local node to let it start\n \t\t\t\t\t\t\t\t  and operate normally even if there is no more nodes in the cluster.\n\t\t\t\t\t\t\t\t\tYou can also optionally specify an individual port or port range.\n \t\t\t\t\t\t  -->\n              <value>1.2.3.4</value>\n              \n              <!-- \n                  IP Address and optional port range of a remote node.\n                  You can also optionally specify an individual port and don't set\n\t\t\t\t\t\t\t\t\tthe port range at all.\n              -->\n              <value>1.2.3.5:47500..47509</value>\n            </list>\n          </property>\n        </bean>\n      </property>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "TcpDiscoverySpi spi = new TcpDiscoverySpi();\n \nTcpDiscoveryVmIpFinder ipFinder = new TcpDiscoveryVmIpFinder();\n \n// Set initial IP addresses.\n// Note that you can optionally specify a port or a port range.\nipFinder.setAddresses(Arrays.asList(\"1.2.3.4\", \"1.2.3.5:47500..47509\"));\n \nspi.setIpFinder(ipFinder);\n \nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default discovery SPI.\ncfg.setDiscoverySpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Multicast and Static IP Based Discovery"
}
[/block]
You can use both, Multicast and Static IP based discovery together. In this case, in addition to addresses received via multicast, if any, `TcpDiscoveryMulticastIpFinder` can also work with pre-configured list of static IP addresses, just like Static IP-Based Discovery described above. Here is an example of how to configure Multicast IP finder with static IP addresses:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"discoverySpi\">\n    <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n      <property name=\"ipFinder\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.multicast.TcpDiscoveryMulticastIpFinder\">\n          <property name=\"multicastGroup\" value=\"228.10.10.157\"/>\n           \n          <!-- list of static IP addresses-->\n          <property name=\"addresses\">\n            <list>\n              <value>1.2.3.4</value>\n              \n              <!-- \n                  IP Address and optional port range.\n                  You can also optionally specify an individual port.\n              -->\n              <value>1.2.3.5:47500..47509</value>\n            </list>\n          </property>\n        </bean>\n      </property>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "TcpDiscoverySpi spi = new TcpDiscoverySpi();\n \nTcpDiscoveryVmIpFinder ipFinder = new TcpDiscoveryMulticastIpFinder();\n \n// Set Multicast group.\nipFinder.setMulticastGroup(\"228.10.10.157\");\n\n// Set initial IP addresses.\n// Note that you can optionally specify a port or a port range.\nipFinder.setAddresses(Arrays.asList(\"1.2.3.4\", \"1.2.3.5:47500..47509\"));\n \nspi.setIpFinder(ipFinder);\n \nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default discovery SPI.\ncfg.setDiscoverySpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Isolated Ignite Clusters on Same Set of Machines"
}
[/block]
There can be a case when you need to start two isolated Ignite clusters on the same set of machines due to testing purposes or by some other reason.

In Ignite this task is achievable if nodes from different clusters will use non intersecting local port ranges for `TcpDiscoverySpi` and `TcpCommunicationSpi`.

Let's say that you need to start two isolated clusters on a single machine for testing purposes.
Then for the nodes from the first cluster, you should use the following `TcpDiscoverySpi` and `TcpCommunicationSpi` configurations:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  \t...\n    <!--\n \t\t\t\tExplicitly configure TCP discovery SPI to provide list of \n\t\t\t\tinitial nodes from the first cluster.\n \t  -->\n    <property name=\"discoverySpi\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n            <!-- Initial local port to listen to. -->\n            <property name=\"localPort\" value=\"48500\"/>\n\n            <!-- Changing local port range. This is an optional action. -->\n            <property name=\"localPortRange\" value=\"20\"/>\n\n            <!-- Setting up IP finder for this cluster -->\n            <property name=\"ipFinder\">\n                <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.vm.TcpDiscoveryVmIpFinder\">\n                    <property name=\"addresses\">\n                        <list>\n                            <!--\n                                Addresses and port range of the nodes from the first\n \t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\tcluster.\n                                127.0.0.1 can be replaced with actual IP addresses or\n \t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\thost names. Port range is optional.\n                            -->\n                            <value>127.0.0.1:48500..48520</value>\n                        </list>\n                    </property>\n                </bean>\n            </property>\n        </bean>\n    </property>\n\n    <!--\n        Explicitly configure TCP communication SPI changing local\n \t\t\t\tport number for the nodes from the first cluster.\n    -->\n    <property name=\"communicationSpi\">\n        <bean class=\"org.apache.ignite.spi.communication.tcp.TcpCommunicationSpi\">\n            <property name=\"localPort\" value=\"48100\"/>\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration cfg=new IgniteConfiguration();\n\n// Explicitly configure TCP discovery SPI to provide list of initial nodes\n// from the first cluster.\nTcpDiscoverySpi discoverySpi=new TcpDiscoverySpi();\n\n// Initial local port to listen to.\ndiscoverySpi.setLocalPort(48500);\n\n// Changing local port range. This is an optional action.\ndiscoverySpi.setLocalPortRange(20);\n\nTcpDiscoveryVmIpFinder ipFinder=new TcpDiscoveryVmIpFinder();\n\n// Addresses and port range of the nodes from the first cluster.\n// 127.0.0.1 can be replaced with actual IP addresses or host names.\n// The port range is optional.\nipFinder.setAddresses(Arrays.asList(\"127.0.0.1:48500..48520\"));\n\n// Overriding IP finder.\ndiscoverySpi.setIpFinder(ipFinder);\n\n// Explicitly configure TCP communication SPI by changing local port number for\n// the nodes from the first cluster.\nTcpCommunicationSpi commSpi=new TcpCommunicationSpi();\n\ncommSpi.setLocalPort(48100);\n\n// Overriding discovery SPI.\ncfg.setDiscoverySpi(discoverySpi);\n\n// Overriding communication SPI.\ncfg.setCommunicationSpi(commSpi);\n\n// Starting a node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
While for the nodes from the second cluster, the configuration can look like this:
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <!--\n        Explicitly configure TCP discovery SPI to provide list of initial\n         nodes from the second cluster.\n    -->\n    <property name=\"discoverySpi\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n            <!-- Initial local port to listen to. -->\n            <property name=\"localPort\" value=\"49500\"/>\n\n            <!-- Changing local port range. This is an optional action. -->\n            <property name=\"localPortRange\" value=\"20\"/>\n\n            <!-- Setting up IP finder for this cluster -->\n            <property name=\"ipFinder\">\n                <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.vm.TcpDiscoveryVmIpFinder\">\n                    <property name=\"addresses\">\n                        <list>\n                            <!--\n                                Addresses and port range of the nodes from the second\n \t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\tcluster.\n                                127.0.0.1 can be replaced with actual IP addresses or\n \t\t\t\t\t\t\t\t\t\t\t\t\t\t\t\thost names. Port range is optional.\n                            -->\n                            <value>127.0.0.1:49500..49520</value>\n                        </list>\n                    </property>\n                </bean>\n            </property>\n        </bean>\n    </property>\n\n    <!--\n        Explicitly configure TCP communication SPI changing local port number \n        for the nodes from the second cluster.\n    -->\n    <property name=\"communicationSpi\">\n        <bean class=\"org.apache.ignite.spi.communication.tcp.TcpCommunicationSpi\">\n            <property name=\"localPort\" value=\"49100\"/>\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration cfg=new IgniteConfiguration();\n\n// Explicitly configure TCP discovery SPI to provide list of initial nodes\n// from the second cluster.\nTcpDiscoverySpi discoverySpi=new TcpDiscoverySpi();\n\n// Initial local port to listen to.\ndiscoverySpi.setLocalPort(49500);\n\n// Changing local port range. This is an optional action.\ndiscoverySpi.setLocalPortRange(20);\n\nTcpDiscoveryVmIpFinder ipFinder=new TcpDiscoveryVmIpFinder();\n\n// Addresses and port range of the nodes from the second cluster.\n// 127.0.0.1 can be replaced with actual IP addresses or host names.\n// The port range is optional.\nipFinder.setAddresses(Arrays.asList(\"127.0.0.1:49500..49520\"));\n\n// Overriding IP finder.\ndiscoverySpi.setIpFinder(ipFinder);\n\n// Explicitly configure TCP communication SPI by changing local port number for\n// the nodes from the second cluster.\nTcpCommunicationSpi commSpi=new TcpCommunicationSpi();\n\ncommSpi.setLocalPort(49100);\n\n// Overriding discovery SPI.\ncfg.setDiscoverySpi(discoverySpi);\n\n// Overriding communication SPI.\ncfg.setCommunicationSpi(commSpi);\n\n// Starting a node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
As you see from the configurations the difference between them is minor - only port numbers for SPIs and IP finder vary.
[block:callout]
{
  "type": "info",
  "body": "If you want the nodes from different clusters are able to look for each other using multicast protocol then replace `TcpDiscoveryVmIpFinder` with `TcpDiscoveryMulticastIpFinder` and set unique `TcpDiscoveryMulticastIpFinder.multicastGroups` in each configuration above."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Apache jclouds Based Discovery"
}
[/block]
Refer to [Generic Cloud Configuration](https://apacheignite-mix.readme.io/docs/jclouds) documentation.
[block:api-header]
{
  "type": "basic",
  "title": "Amazon S3 Based Discovery"
}
[/block]
Refer to [AWS Configuration](https://apacheignite-mix.readme.io/docs/amazon-aws) documentation.
[block:api-header]
{
  "type": "basic",
  "title": "Google Cloud Storage Based Discovery"
}
[/block]
Refer to [Google Cloud Configuration](https://apacheignite-mix.readme.io/docs/google-compute-engine) documentation.
[block:api-header]
{
  "type": "basic",
  "title": "JDBC Based Discovery"
}
[/block]
You can have your database be a common shared storage of initial IP addresses. With this IP finder nodes will write their IP addresses to a database on startup. This is done via `TcpDiscoveryJdbcIpFinder`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"discoverySpi\">\n    <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n      <property name=\"ipFinder\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.jdbc.TcpDiscoveryJdbcIpFinder\">\n          <property name=\"dataSource\" ref=\"ds\"/>\n        </bean>\n      </property>\n    </bean>\n  </property>\n</bean>\n\n<!-- Configured data source instance. -->\n<bean id=\"ds\" class=\"some.Datasource\">\n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "TcpDiscoverySpi spi = new TcpDiscoverySpi();\n\n// Configure your DataSource.\nDataSource someDs = MySampleDataSource(...);\n\nTcpDiscoveryJdbcIpFinder ipFinder = new TcpDiscoveryJdbcIpFinder();\n\nipFinder.setDataSource(someDs);\n\nspi.setIpFinder(ipFinder);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default discovery SPI.\ncfg.setDiscoverySpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Shared File System Based Discovery"
}
[/block]
A shared file system can be used as a storage for nodes' IP addresses. The nodes will write their IP addresses to the file system on startup. This behavior is supported by `TcpDiscoverySharedFsIpFinder`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"discoverySpi\">\n    <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n      <property name=\"ipFinder\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.sharedfs.TcpDiscoverySharedFsIpFinder\">\n          <property name=\"path\" ref=\"/var/ignite/addresses\"/>\n        </bean>\n      </property>\n    </bean>\n  </property>\n</bean>",
      "language": "xml",
      "name": "XML"
    },
    {
      "code": "// Configuring discovery SPI.\nTcpDiscoverySpi spi = new TcpDiscoverySpi();\n\n// Configuring IP finder.\nTcpDiscoverySharedFsIpFinder ipFinder = new TcpDiscoverySharedFsIpFinder();\n\nipFinder.setPath(\"/var/ignite/addresses\");\n\nspi.setIpFinder(ipFinder);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default discovery SPI.\ncfg.setDiscoverySpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Kubernetes Service Based Discovery"
}
[/block]
Refer to [Kubernetes Discovery](https://apacheignite-mix.readme.io/docs/kubernetes-discovery) documentation.
[block:api-header]
{
  "type": "basic",
  "title": "ZooKeeper Based Discovery"
}
[/block]
If you're using [ZooKeeper](https://zookeeper.apache.org/) to coordinate your distributed environment, you can utilize it for Ignite nodes discovery as well. This is done via `TcpDiscoveryZookeeperIpFinder` (note that `ignite-zookeeper` module has to be enabled).
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"discoverySpi\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n            <property name=\"ipFinder\">\n                <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.zk.TcpDiscoveryZookeeperIpFinder\">\n                    <property name=\"zkConnectionString\" value=\"127.0.0.1:2181\"/>\n                </bean>\n            </property>\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "TcpDiscoverySpi spi = new TcpDiscoverySpi();\n\nTcpDiscoveryZookeeperIpFinder ipFinder = new TcpDiscoveryZookeeperIpFinder();\n\n// Specify ZooKeeper connection string.\nipFinder.setZkConnectionString(\"127.0.0.1:2181\");\n\nspi.setIpFinder(ipFinder);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default discovery SPI.\ncfg.setDiscoverySpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Failure Detection Timeout"
}
[/block]
Failure detection timeout is used to determine how long a cluster node should wait before considering a remote connection, with another node, failed. 

Every node in the Ignite cluster is connected with another node, at the level of discovery SPI. NodeA sends heartbeats (and other system messages transferred over the cluster ring - discovery SPI) to nodeB, and if the latter doesn’t reply in `failureDetectionTimeout`, then nodeB will be kicked off the cluster.

This timeout is the easiest way to tune discovery SPI's failure detection feature depending on the network and hardware conditions of your environment.
[block:callout]
{
  "type": "warning",
  "body": "The timeout automatically controls configuration parameters of `TcpDiscoverySpi`, such as socket timeout, message acknowledgment timeout and others. If any of these parameters is set explicitly, then the failure timeout setting will be ignored.",
  "title": ""
}
[/block]
The failure detection timeout is configured using `IgniteConfiguration.setFailureDetectionTimeout(long)` for Apache Ignite server nodes and `IgniteConfiguration.setClientFailureDetectionTimeout(long)` for client nodes. The default value, that is equal to 10 seconds for the server nodes and 30 seconds for the client nodes. This allows the discovery SPI to work reliably on most of the on-premise and containerized deployments. However, for stable low-latency networks, the parameter can be set to ~200 milliseconds in order to detect and react to​ failures more quickly.       
[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
Below you can see the most frequently used `TcpDiscoverySpi` configuration parameters. Refer to `TcpDiscoverySpi` javadoc to see the full list of configuration options.
[block:parameters]
{
  "data": {
    "0-0": "`setIpFinder(TcpDiscoveryIpFinder)`",
    "0-1": "IP finder that is used to share info about nodes IP addresses.",
    "0-2": "`TcpDiscoveryMulticastIpFinder`\n\nSome of the implementations that can be used:\n`TcpDiscoverySharedFsIpFinder`\n`TcpDiscoveryS3IpFinder`\n`TcpDiscoveryJdbcIpFinder`\n`TcpDiscoveryVmIpFinder`",
    "h-0": "Setter Method",
    "h-1": "Description",
    "h-2": "Default",
    "h-3": "Default",
    "0-3": "",
    "1-0": "`setLocalAddress(String)`",
    "1-1": "Sets local host IP address that discovery SPI uses.",
    "1-3": "",
    "1-2": "If not provided, by default a first found non-loopback address will be used. If there is no non-loopback address available, then `java.net.InetAddress.getLocalHost()` will be used.",
    "2-0": "`setLocalPort(int)`",
    "2-1": "Port the SPI listens to.",
    "2-2": "47500",
    "2-3": "",
    "3-0": "`setLocalPortRange(int)`",
    "3-1": "Local port range. \nLocal node will try to bind on first available port starting from local port up until local port + local port range.",
    "3-2": "100",
    "3-3": "100",
    "4-0": "`setReconnectCount(int)`",
    "4-1": "Number of times node tries to (re)establish connection to another node.",
    "4-3": "2",
    "4-2": "2",
    "5-0": "`setNetworkTimeout(long)`",
    "5-1": "Sets maximum network timeout in milliseconds to use for network operations.",
    "5-2": "5000",
    "5-3": "5000",
    "6-0": "`setSocketTimeout(long)`",
    "6-1": "Sets socket operations timeout. This timeout is used to limit connection time and write-to-socket time.",
    "6-2": "2000",
    "6-3": "2000",
    "7-0": "`setAckTimeout(long)`",
    "7-1": "Sets timeout for receiving acknowledgement for sent message. \nIf acknowledgement is not received within this timeout, sending is considered as failed and SPI tries to repeat message sending.",
    "7-2": "2000",
    "7-3": "2000",
    "8-0": "`setJoinTimeout(long)`",
    "8-1": "Sets join timeout. If non-shared IP finder is used and node fails to connect to any address from IP finder, node keeps trying to join within this timeout. If all addresses are still unresponsive, exception is thrown and node startup fails. \n0 means wait forever.",
    "8-2": "0",
    "8-3": "0",
    "9-0": "`setThreadPriority(int)`",
    "9-1": "Thread priority for threads started by SPI.",
    "9-2": "0",
    "9-3": "0",
    "10-0": "`setStatisticsPrintFrequency(int)`",
    "10-1": "Statistics print frequency in milliseconds. \n0 indicates that no print is required. If value is greater than 0 and log is not quiet then stats are printed out with INFO level once a period. This may be very helpful for tracing topology problems.",
    "10-2": "true",
    "10-3": "true",
    "11-0": ""
  },
  "cols": 3,
  "rows": 11
}
[/block]