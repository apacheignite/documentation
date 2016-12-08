* [Overview](#overview) 
* [IgniteServices](#igniteservices)
* [Load Balancing](#load-balancing)
* [Fault Tolerance](#fault-tolerance)
* [Deployment Management](#deployment-management)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Service Grid allows for deployments of arbitrary user-defined services on the cluster. You can implement and deploy any service, such as custom counters, ID generators, hierarchical maps, etc.

Ignite allows you to control how many instances of your service should be deployed on each cluster node and will automatically ensure proper deployment and fault tolerance of all the services . 
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/qOINezbjTiOJFl3ueIUs_ignite_service.png",
        "ignite_service.png",
        "1645",
        "741",
        "#856557",
        ""
      ]
    }
  ]
}
[/block]
##Features
  * **Continuous availability** of deployed services regardless of topology changes or crashes.
  * Automatically deploy any number of distributed service instances in the cluster.
  * Automatically deploy [singletons](doc:cluster-singletons), including cluster-singleton, node-singleton, or key-affinity-singleton.
  * Automatically deploy distributed services on node start-up by specifying them in the  configuration.
  * Undeploy any of the deployed services.
  * Get information about service deployment topology within the cluster.
  * Create service proxy for accessing remotely deployed distributed services.
[block:callout]
{
  "type": "success",
  "body": "Please refer to [Service Example](doc:service-example) for information on service deployment and accessing service API."
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Note, that by default it's required to have a Service class in the classpath of all the cluster nodes. [Peer class loading](doc:zero-deployment) (P2P class loading) is not supported for Service Grid. Learn more from [services deployment management](https://apacheignite.readme.io/docs/service-grid#deployment-management) section on how to overcome this default requirement."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "IgniteServices"
}
[/block]
All service grid functionality is available via `IgniteServices` interface.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\n// Get services instance spanning all nodes in the cluster.\nIgniteServices svcs = ignite.services();",
      "language": "java"
    }
  ]
}
[/block]
You can also limit the scope of service deployment to a Cluster Group. In this case, services will only span the nodes within the cluster group.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignitition.ignite();\n\nClusterGroup remoteGroup = ignite.cluster().forRemotes();\n\n// Limit service deployment only to remote nodes (exclude the local node).\nIgniteServices svcs = ignite.services(remoteGroup);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Load Balancing"
}
[/block]
In all cases, other than singleton service deployment, Ignite will automatically make sure that about an equal number of services are deployed on each node within the cluster. Whenever cluster topology changes, Ignite will re-evaluate service deployments and may re-deploy an already deployed service on another node for better load balancing.
[block:api-header]
{
  "type": "basic",
  "title": "Fault Tolerance"
}
[/block]
Ignite always guarantees that services are continuously available, and are deployed according to the specified configuration, regardless of any topology changes or node crashes.
[block:api-header]
{
  "type": "basic",
  "title": "Deployment Management"
}
[/block]
By default, an Ignite Service will be deployed on a random node (or nodes) depending on the cluster workload as it's described in the load balancing section above.

In addition to this default approach, Ignite provides an API that allows limiting an Ignite Service deployment to a specific set of nodes. There are several approaches to do that and all of them are covered below.

##Node Filter Based Deployment

This approach is based on a filtering predicate that gets called on every node at the time Ignite Service engine determines a set of possible candidates for the Ignite Service deployment. If the predicate returns `true` then a node will be included in the set or it will be excluded otherwise.

A node filter has to implement `IgnitePredicate<ClusterNode>` interface like it does the exemplary filter below which will instruct the Service Grid engine to deploy an Ignite Service on non client nodes that have `west.coast.attribute` in their local attributes map.

[block:code]
{
  "codes": [
    {
      "code": "// The filter implementation.\npublic class ServiceFilter implements IgnitePredicate<ClusterNode> {\n\t@Override public boolean apply(ClusterNode node) {\n  \t// The service will be deployed on non client nodes\n    // that have the attribute 'west.coast.node'.\n    return !node.isClient() &&\n    node.attributes().containsKey(\"west.coast.node\");\n  }\n}",
      "language": "java"
    }
  ]
}
[/block]
After the filter is ready you can pass it to `ServiceConfiguration.setNodeFilter(...)` method and start the service using this configuration.
[block:code]
{
  "codes": [
    {
      "code": "// Initiating cache configuration. \nServiceConfiguration cfg = new ServiceConfiguration();\n        \n// Setting service instance to deploy.\ncfg.setService(service);\n        \n// Setting service name.\ncfg.setName(\"serviceName\");\n        \n// Providing the nodes filter.\ncfg.setNodeFilter(new ServiceFilter());\n        \n// Getting instance of Ignite Service Grid.\nIgniteServices services = ignite.services();\n        \n// Deploying the service.\nservices.deploy(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Make sure that a node filter's class is located in the classpath of every Ignite node regardless of the fact whether an Ignite Service is going to be deployed there or not. Otherwise you'll get a `ClassNotFoundException`.\n\nOn the other hand, you're free to add the Ignite Service class implementation to the classpath of those nodes only where the service might be deployed in principle throughout the lifetime of a cluster."
}
[/block]
## Cluster Group Based Deployment

One more approach is based on the definition of a specific `ClusterGroup`. Once a reference to Ignite Service Grid is obtained for a specific cluster group the deployment of a new Ignite Service will happen on one or a number of the nodes from that group only.
[block:code]
{
  "codes": [
    {
      "code": "// A service will be deployed on the server nodes only.\nIgniteServices services = ignite.services(ignite.cluster().forServers());\n\n// Deploying the service.\nservices.deploy(serviceCfg);\n            ",
      "language": "java"
    }
  ]
}
[/block]
## Affinity Key Based Deployment

The latest approach that allows influencing on an Ignite Service deployment is based on an affinity key definition. The service's configuration has to contain a value for the affinity key as well as a cache name to which this key belongs to and during the service startup Ignite Service Grid will deploy the service on a node that is primary for the given key. If the primary node changes throughout the time then the service will be re-deployed automatically as well.
[block:code]
{
  "codes": [
    {
      "code": "// Initiating cache configuration.\nServiceConfiguration cfg = new ServiceConfiguration();\n\n// Setting service instance to deploy.\ncfg.setService(service);\n\n// Setting service name.\ncfg.setName(\"serviceName\");\n\n// Setting the cache name and key's value for the affinity based deployment.\ncfg.setCacheName(\"orgCache\");\ncfg.setAffinityKey(123);\n\n// Getting instance of Ignite Service Grid.\nIgniteServices services = ignite.services();\n\n// Deploying the service.\nservices.deploy(cfg);",
      "language": "java"
    }
  ]
}
[/block]
After the service is deployed using the configuration from the example above, Ignite will make sure to deploy the service on a node that is primary for key `123` stored in cache named `orgCache`.