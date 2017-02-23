* [IgniteCluster](#ignitecluster)
* [ClusterNode](#clusternode)
* [Cluster Node Attributes](#cluster-node-attributes)
* [Cluster Node Metrics](#cluster-node-metrics)
* [Local Cluster Node](#local-cluster-node)
[block:api-header]
{
  "type": "basic",
  "title": "IgniteCluster"
}
[/block]
Cluster functionality is provided via the `IgniteCluster` interface. You can get an instance of `IgniteCluster` from `Ignite` as follows:
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nIgniteCluster cluster = ignite.cluster();",
      "language": "java"
    }
  ]
}
[/block]
Through the `IgniteCluster` interface you can:
 * Start and stop remote cluster nodes
 * Get a list of all cluster members
 * Create logical [Cluster Groups](doc:cluster-groups)
[block:api-header]
{
  "type": "basic",
  "title": "ClusterNode"
}
[/block]
The `ClusterNode` interface has a very concise API and deals only with the node as a logical network endpoint in the topology: its globally unique ID, the node metrics, its static attributes set by the user and a few other parameters.
[block:api-header]
{
  "type": "basic",
  "title": "Cluster Node Attributes"
}
[/block]
All cluster nodes on startup automatically register all the environment and system properties as node attributes. However, users can choose to assign their own node attributes in the configuration:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.IgniteConfiguration\">\n    ...\n    <property name=\"userAttributes\">\n        <map>\n            <entry key=\"ROLE\" value=\"worker\"/>\n        </map>\n    </property>\n    ...\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
The following example shows how to get the nodes where "worker" attribute has been set.
[block:code]
{
  "codes": [
    {
      "code": "ClusterGroup workers = ignite.cluster().forAttribute(\"ROLE\", \"worker\");\n\nCollection<ClusterNode> nodes = workers.nodes();",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "success",
  "body": "All node attributes are available via `ClusterNode.attribute(\"propertyName\")` method."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Cluster Node Metrics"
}
[/block]
Ignite automatically collects metrics for all the nodes in the cluster. Metrics are collected in the background and are updated with every heartbeat message exchanged between the cluster nodes.

Node metrics are available via the `ClusterMetrics` interface which contains over 50 various metrics (note that the same metrics are available for [Cluster Groups](doc:cluster-groups)  as well).

Here is an example of getting some metrics, including average CPU load and used heap, for the local node:
[block:code]
{
  "codes": [
    {
      "code": "// Local Ignite node.\nClusterNode localNode = cluster.localNode();\n\n// Node metrics.\nClusterMetrics metrics = localNode.metrics();\n\n// Get some metric values.\ndouble cpuLoad = metrics.getCurrentCpuLoad();\nlong usedHeap = metrics.getHeapMemoryUsed();\nint numberOfCores = metrics.getTotalCpus();\nint activeJobs = metrics.getCurrentActiveJobs();",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Local Cluster Node"
}
[/block]
The local grid node is an instance of the `ClusterNode` representing *this* Ignite node. 

Here is an example of how to get a local node:
[block:code]
{
  "codes": [
    {
      "code": "ClusterNode localNode = ignite.cluster().localNode();",
      "language": "java"
    }
  ]
}
[/block]