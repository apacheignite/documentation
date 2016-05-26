With [aspect-oriented-programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming) you can implicitly change any method's behavior (for example, add logging or make a method transactional). In Ignite, you can turn your method into a closure which will run on the grid. This is as simple as adding a `@Gridify` annotation to a method:
[block:code]
{
  "codes": [
    {
      "code": "@Gridify\npublic void sayHello() {\n    System.out.println(\"Hello!\");\n}",
      "language": "java"
    }
  ]
}
[/block]
You can use any AOP library of your choice: **AspectJ**, **JBoss**, or **Spring AOP**. Provided that you have properly configured one of these frameworks for your master node (node which will originate the execution), a mere invocation of the above method would result in a task execution across the grid, which will print "Hello!" on all worker nodes in your topology.
[block:api-header]
{
  "type": "basic",
  "title": "Gridify Annotation"
}
[/block]
`@Gridify` annotation can be used on any method, and has the following parameters, all of which are optional:
[block:parameters]
{
  "data": {
    "0-0": "taskClass",
    "0-1": "Class",
    "0-2": "Custom task class.",
    "0-3": "`GridifyDefaultTask.class`",
    "h-0": "Parameter Name",
    "h-1": "Type",
    "h-2": "Description",
    "h-3": "Default Value",
    "1-0": "taskName",
    "1-1": "String",
    "1-2": "Name of the custom task to launch.",
    "1-3": "\" \"",
    "2-0": "timeout",
    "2-1": "long",
    "2-2": "Task execution timeout. 0 for no timeout.",
    "2-3": "0",
    "3-0": "interceptor",
    "3-1": "Class",
    "3-2": "Interceptor class for filtering gridified methods.",
    "3-3": "`GridifyInterceptor.class`",
    "4-0": "gridName\tString",
    "4-1": "String",
    "4-2": "Name of the grid to use.",
    "4-3": "\" \""
  },
  "cols": 4,
  "rows": 5
}
[/block]
Generally, if you call a *Gridified* method, the following happens:
  * A grid with specified `gridName` will be used for execution (if no grid name is specified, default no-name grid will be used).
  * If specified, an interceptor is used to check if the method should be grid-enabled. If interceptor returns `false`, a method is called as usual, without grid-enabling.
  * A grid task is created and executed with effective method arguments, `this` object (if method is non-static), and timeout.
  * The return value of grid task is returned from the *Gridified* method. 
 
## Default Behaviour 
If you use @Gridify annotation with no parameters, the default behaviour is implied, which is the following:
  * A task of class `GridifyDefaultTask` is created, which generates 1 job of class `GridifyJobAdapter`, and uses default load balancer for choosing a worker node.
  * A job on remote node invokes a method with the passed-in parameters, using deserialized this object (or null if the method is static), and returns the method result as job result.
  * The job result on remote node will become a task result on the caller side.
  * Task result will be returned to user as a *Gridified* method return value. 
 
##Custom Task
You can use a custom task for specifying grid-enabling logic for a *gridified* method. An example below broadcasts the *gridified* method to all available worker nodes and skips the reduce step (meaning that this task returns nothing):
[block:code]
{
  "codes": [
    {
      "code": "// Custom task class.\nclass GridifyBroadcastMethodTask extends GridifyTaskAdapter<Void> {\n    @Nullable @Override\n    public Map<? extends ComputeJob, ClusterNode> map(List<ClusterNode> subgrid, @Nullable GridifyArgument arg) throws IgniteException {\n        Map<ComputeJob, ClusterNode> ret = new HashMap<>(subgrid.size());\n\n        // Broadcast method to all nodes.\n        for (ClusterNode node : subgrid)\n            ret.put(new GridifyJobAdapter(arg), node);\n\n        return ret;\n    }\n\n    @Nullable @Override\n    public Void reduce(List<ComputeJobResult> list) throws IgniteException {\n        return null; // No-op.\n    }\n}\n \npublic class GridifyHelloWorldTaskExample {\n\t...\n\t// Gridified method. \n\t@Gridify(taskClass = GridifyBroadcastMethodTask.class)\n\tpublic static void sayHello(String arg) {\n\t    System.out.println(\"Hello, \" + arg + '!');\n\t}\n\t...\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring AOP"
}
[/block]
Ignite supports 3 AOP frameworks:
  * AspectJ
  * JBoss AOP
  * Spring AOP

In this section we describe the configuration details for each of the above frameworks. We assume IGNITE_HOME to be an Ignite installation directory.

[block:callout]
{
  "type": "warning",
  "body": "These details apply to a logical master node only (the node that initiates the execution). Logical worker nodes should be left intact."
}
[/block]
**AspectJ**

To enable AspectJ byte code weaving, your master node's JVM should be configured the following way:
  * It should be launched with `-javaagent:IGNITE_HOME/libs/aspectjweaver-1.8.9.jar` argument.
  * The classpath should contain `IGNITE_HOME/config/aop/aspectj` folder.
 
**JBoss AOP**
To enable JBoss byte code weaving, your master node's JVM should have the following configuration:
  * It should be launched with the arguments:
    * -javaagent:[path to jboss-aop-jdk50-4.x.x.jar]
    * -Djboss.aop.class.path=[path to ignite.jar]
    * -Djboss.aop.exclude=org,com
    * -Djboss.aop.include=[your package name]
    
  * It should contain the following jars in classpath:
     * javassist-3.x.x.jar
     * jboss-aop-jdk50-4.x.x.jar
     * jboss-aspect-library-jdk50-4.x.x.jar
     * jboss-common-4.x.x.jar
     * trove-1.0.2.jar
[block:callout]
{
  "type": "warning",
  "body": "Ignite is not shipped with JBoss, and necessary libraries will have to be downloaded separately (they come standard if you have JBoss installed already).",
  "title": "JBoss Dependencies"
}
[/block]
**Spring AOP**

Spring AOP framework is based on dynamic proxy implementation and doesn't require any specific runtime parameters for online weaving. All weaving is on-demand and should be performed by calling method `GridifySpringEnhancer.enhance()` for the object that has method with `@Gridify` annotation.
Note that this method of weaving is rather inconvenient and AspectJ or JBoss AOP are recommended instead. Spring AOP can be used in situation when code augmentation is undesired and cannot be used. It also allows for very fine grained control of what gets weaved.