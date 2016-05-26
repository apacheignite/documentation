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
    "0-3": "GridifyDefaultTask.class",
    "h-0": "Parameter Name",
    "h-1": "Type",
    "h-2": "Description",
    "h-3": "Default Value"
  },
  "cols": 4,
  "rows": 4
}
[/block]