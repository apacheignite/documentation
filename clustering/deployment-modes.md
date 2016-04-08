Specifics of peer-class-loading behavior are controlled by different deployment modes. Particularly, the un-deployment behavior in cases when originating node leaves grid depends on the deployment mode. Other aspects, governed by deployment mode, are user resources management and class versions management. In the sections below we describe each deployment mode in more detail.
[block:api-header]
{
  "type": "basic",
  "title": "`PRIVATE`"
}
[/block]
In this mode deployed classes do not share user resources (see Resource Injection).
Basically, user resources are created once per deployed task class, and then get reused for all executions. Note that classes deployed within the same class loader on master node, will still share the same class loader remotely on worker nodes. However, tasks deployed from different master nodes will not share the same class loader on worker nodes, which is useful in development when different developers can be working on different versions of the same classes. Also note that resources are associated with task deployment, not task execution. If the same deployed task gets executed multiple times, then it will keep reusing the same user resources every time.

In this mode classes are un-deployed when master node leaves the grid.
[block:api-header]
{
  "type": "basic",
  "title": "`ISOLATED`"
}
[/block]
Unlike `PRIVATE` mode, where different deployed tasks will never use the same instance of user resources, in `ISOLATED` mode, tasks or classes deployed within the same class loader will share the same instances of user resources (see Resource Injection). This means that if multiple tasks classes are loaded by the same class loader on master node, then they will share instances of user resources on worker nodes. In other words, user resources get initialized once per class loader and then get reused for all consequent executions. Note, that classes deployed within the same class loader on master node, will still share the same class loader remotely on worker nodes. However, tasks deployed from different master nodes will not share the same class loader on worker nodes, which is especially useful when different developers can be working on different versions of the same classes.

In this mode classes are un-deployed when master node leaves the grid.
[block:api-header]
{
  "type": "basic",
  "title": "`SHARED`"
}
[/block]
This is the default deployment mode. In this mode, classes from different master nodes with the same user version will share the same class loader on worker nodes. Classes will be un-deployed whenever all master nodes leave grid or user version changes. This mode allows classes coming from different master nodes share the same instances of user resources on remote nodes (see below). This method is specifically useful in production as, in comparison to `ISOLATED` mode, which has a scope of single class loader on a single master node, `SHARED` mode broadens the deployment scope to all master nodes.
[block:api-header]
{
  "type": "basic",
  "title": "`CONTINUOUS`"
}
[/block]
In `CONTINUOUS` mode, the classes are not un-deployed when master nodes leave grid. Un-deployment only happens when a class user version changes. The advantage of this approach is that it allows tasks coming from different master nodes share the same instances of user resources (see Resource Injection) on worker nodes. This allows for all tasks executing on worker nodes to reuse, for example, the same instances of connection pools or caches. When using this mode, you can startup multiple stand-alone GridGain worker nodes, define user resources on master nodes and have them initialized once on worker nodes regardless of which master node they came from. In comparison to `ISOLATED` deployment mode which has a scope of single class loader on a single master node, `CONTINUOUS` mode broadens the deployment scope to all master nodes which is specifically useful in production.
[block:api-header]
{
  "type": "basic",
  "title": "Un-Deployment and User Versions"
}
[/block]
The class definitions, obtained with peer class loading, have their own lifecycle. On certain events (when master node leaves or user version changes, depending on deployment mode), the class information is un-deployed from the grid: the class definition is erased from all nodes in the grid and the user resources, linked with that class definition, are also optionally erased (again, depending on deployment mode). For In-Memory Data Grid, it also means that all cache entries of an un-deployed class are removed from cache.
User version comes into play whenever you would like to redeploy classes deployed in `SHARED` or `CONTINUOUS` modes. By default, Ignite will automatically detect if class-loader has changed or a node is restarted. However, if you would like to change and redeploy code on a subset of nodes, or in case of CONTINUOUS mode,  kill every living deployment, you should change the user version.
User version is specified in META-INF/gridgain.xml file of your classpath as follows:
[block:code]
{
  "codes": [
    {
      "code": "<!-- User version. -->\n<bean id=\"userVersion\" class=\"java.lang.String\">\n    <constructor-arg value=\"0\"/>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
By default, all Ignite startup scripts (ignite.sh or ignite.bat) pick up user version from IGNITE_HOME/config/userversion folder. Usually, it is just enough to update user version under that folder. However, in case of GAR or JAR deployment, you should remember to provide META-INF/ignite.xml file with the desired user version in it.
[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
All peer class loading configuration options are available in GridConfiguration. Below is a table enlisting these options: