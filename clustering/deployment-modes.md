Specifics of peer-class-loading behavior are controlled by different deployment modes. Particularly, the un-deployment behavior in cases when originating node leaves grid depends on the deployment mode. Other aspects, governed by deployment mode, are user resources management and class versions management. In the sections below we describe each deployment mode in more detail.
[block:api-header]
{
  "type": "basic",
  "title": "PRIVATE"
}
[/block]
In this mode deployed classes do not share user resources (see Resource Injection).
Basically, user resources are created once per deployed task class, and then get reused for all executions. Note that classes deployed within the same class loader on master node, will still share the same class loader remotely on worker nodes. However, tasks deployed from different master nodes will not share the same class loader on worker nodes, which is useful in development when different developers can be working on different versions of the same classes. Also note that resources are associated with task deployment, not task execution. If the same deployed task gets executed multiple times, then it will keep reusing the same user resources every time.

In this mode classes are un-deployed when master node leaves the grid.
[block:api-header]
{
  "type": "basic",
  "title": "ISOLATED"
}
[/block]
Unlike `PRIVATE` mode, where different deployed tasks will never use the same instance of user resources, in `ISOLATED` mode, tasks or classes deployed within the same class loader will share the same instances of user resources (see Resource Injection). This means that if multiple tasks classes are loaded by the same class loader on master node, then they will share instances of user resources on worker nodes. In other words, user resources get initialized once per class loader and then get reused for all consequent executions. Note, that classes deployed within the same class loader on master node, will still share the same class loader remotely on worker nodes. However, tasks deployed from different master nodes will not share the same class loader on worker nodes, which is especially useful when different developers can be working on different versions of the same classes.

In this mode classes are un-deployed when master node leaves the grid.
[block:api-header]
{
  "type": "basic",
  "title": "SHARED"
}
[/block]
This is the default deployment mode. In this mode, classes from different master nodes with the same user version will share the same class loader on worker nodes. Classes will be un-deployed whenever all master nodes leave grid or user version changes. This mode allows classes coming from different master nodes share the same instances of user resources on remote nodes (see below). This method is specifically useful in production as, in comparison to `ISOLATED` mode, which has a scope of single class loader on a single master node, `SHARED` mode broadens the deployment scope to all master nodes.
[block:api-header]
{
  "type": "basic",
  "title": "CONTINUOUS"
}
[/block]
In `CONTINUOUS` mode, the classes are not un-deployed when master nodes leave grid. Un-deployment only happens when a class user version changes. The advantage of this approach is that it allows tasks coming from different master nodes share the same instances of user resources (see Resource Injection) on worker nodes. This allows for all tasks executing on worker nodes to reuse, for example, the same instances of connection pools or caches. When using this mode, you can startup multiple stand-alone GridGain worker nodes, define user resources on master nodes and have them initialized once on worker nodes regardless of which master node they came from. In comparison to `ISOLATED` deployment mode which has a scope of single class loader on a single master node, `CONTINUOUS` mode broadens the deployment scope to all master nodes which is specifically useful in production.