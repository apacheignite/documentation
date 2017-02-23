[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite cluster can be easily deployed in and maintained by [Kubernetes](https://kubernetes.io) which is an open-source system for automating deployment, scaling, and management of containerized applications.

This getting started guide walks you through the actions needed to be done if plan to deploy an Apache Ignite cluster in Kubernetes environment.
[block:api-header]
{
  "type": "basic",
  "title": "Kubernetes IP finder"
}
[/block]
To enable Apache Ignite nodes auto-discovery in Kubernetes you need to enable `TcpDiscoveryKubernetesIpFinder` in the nodes' cluster configuration. Let's introduce this configuration by naming it as `example-kube.xml` and defining as follows:
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\n        http://www.springframework.org/schema/util\n        http://www.springframework.org/schema/util/spring-util.xsd\">\n\n<bean id=\"ignite.cfg\"\n    class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <property name=\"discoverySpi\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n        \t\t<property name=\"ipFinder\">\n            \t\t<bean\nclass=\"org.apache.ignite.spi.discovery.tcp.ipfinder.kubernetes.TcpDiscoveryKubernetesIpFinder\">\n            \t\t</bean>\n            </property>\n        </bean>\n    </property>\n</bean>\n</beans>\n",
      "language": "xml",
      "name": "example-kube.xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Kubernetes Ignite Lookup Service"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Sharing Ignite Cluster Configuration"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Ignite Pods Deployment"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Adjusting Ignite Cluster Size"
}
[/block]