--------------
title: Service Grid
excerpt: Cluster-enable any service or data structure.
--------------

Service Grid allows for deployments of arbitrary user-defined services on the cluster. You can implement and deploy any service, such as custom counters, ID generators, hierarchical maps, etc.

Ignite allows you to control how many instances of your service should be deployed on each cluster node and will automatically ensure proper deployment and fault tolerance of all the services . 
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/c3Rr8tQ1RGyRw4cxMVoD",
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