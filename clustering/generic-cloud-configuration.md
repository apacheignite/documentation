* [Overview](#overview)
* [Apache JClouds Based Discovery](#apache-jclouds-based-discovery)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Nodes discovery on a cloud platform is usually proven to be more challenging because the most of of such virtual environments, has the following limitations:
* Multicast is disabled;
* TCP addresses change every time a new image is started.

Although you can use TCP-based discovery in the absence of the Multicast, you still have to deal with constantly changing IP addresses and constantly updating the configuration. This creates a major inconvenience and makes configurations based on static IPs virtually unusable in such environments.
[block:api-header]
{
  "type": "basic",
  "title": "Apache JClouds Based Discovery"
}
[/block]
To mitigate constantly changing IP addresses problem, Ignite supports automatic node discovery by utilizing Apache jclouds multi-cloud toolkit via `TcpDiscoveryCloudIpFinder`. 
For information about Apache jclouds please refer to [jclouds.apache.org](https://jclouds.apache.org).

The IP finder forms nodes addresses, that possibly running Apache Ignite, by getting private and public IP addresses of all virtual machines running on a cloud and adding a port number to them. The port is either the one that is set with 'TcpDiscoverySpi.setLocalPort(int)' or 'TcpDiscoverySpi.DFLT_PORT' otherwise.
This way all the nodes can try to connect to any formed IP address and initiate automatic grid node discovery.

Please refer to [Apache jclouds providers section](https://jclouds.apache.org/reference/providers/#compute) to get the list of supported cloud platforms.
[block:callout]
{
  "type": "warning",
  "body": "All virtual machines must start Ignite instances on the same port, otherwise they will not be able to discover each other using this IP finder."
}
[/block]
Here is an example of how to configure Apache jclouds based IP finder:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"discoverySpi\">\n    <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n      <property name=\"ipFinder\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.cloud.TcpDiscoveryCloudIpFinder\">\n        \t<!-- Configuration for Google Compute Engine. -->\n        \t<property name=\"provider\" value=\"google-compute-engine\"/>\n        \t<property name=\"identity\" value=\"YOUR_SERVICE_ACCOUNT_EMAIL\"/>\n        \t<property name=\"credentialPath\" value=\"PATH_YOUR_PEM_FILE\"/>\n        \t<property name=\"zones\">\n          \t<list>\n            \t<value>us-central1-a</value>\n            \t<value>asia-east1-a</value>\n          \t</list>\n        \t</property>\n        </bean>\n      </property>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "TcpDiscoverySpi spi = new TcpDiscoverySpi();\n\nTcpDiscoveryCloudIpFinder ipFinder = new TcpDiscoveryCloudIpFinder();\n\n// Configuration for AWS EC2.\nipFinder.setProvider(\"aws-ec2\");\nipFinder.setIdentity(yourAccountId);\nipFinder.setCredential(yourAccountKey);\nipFinder.setRegions(Collections.<String>emptyList().add(\"us-east-1\"));\nipFinder.setZones(Arrays.asList(\"us-east-1b\", \"us-east-1e\"));\n\nspi.setIpFinder(ipFinder);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default discovery SPI.\ncfg.setDiscoverySpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Refer to [Cluster Configuration](doc:cluster-config) for more information on various cluster configuration properties."
}
[/block]