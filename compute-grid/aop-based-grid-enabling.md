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
You can use any AOP library of your choice: AspectJ, JBoss, or Spring AOP. Provided that you have properly configured one of these frameworks for your master node (node which will originate the execution), a mere invocation of the above method would result in a task execution across the grid, which will print "Hello!" on all worker nodes in your topology.
[block:api-header]
{
  "type": "basic",
  "title": "Gridify Annotation Parameters"
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
Generally, if you call a Gridified method, the following happens:
  * A grid with specified `gridName` will be used for execution (if no grid name is specified, default no-name grid will be used).
  * If specified, an interceptor is used to check if the method should be grid-enabled. If interceptor returns `false`, a method is called as usual, without grid-enabling.
  * A grid task is created and executed with effective method arguments, `this` object (if method is non-static), and timeout.
  * The return value of grid task is returned from *Gridified* method. 
 
## Default Behaviour 
If you use @Gridify annotation with no parameters, the default behaviour is implied, which is the following:
  * A task of class `GridifyDefaultTask` is created, which generates 1 job of class `GridifyJobAdapter`, and uses default load balancer for choosing a worker node.
  * A job on remote node invokes a method with the passed-in parameters, using deserialized this object (or null if the method is static), and returns the method result as job result.
  * The job result on remote node will become a task result on the caller side.
  * Task result will be returned to user as *Gridified* method return value.