* [Custom Thread Pools](#section-custom-thread-pools)
[block:api-header]
{
  "title": "Custom Thread Pools"
}
[/block]
It is possible to configure a custom thread pool for Ignite Compute tasks. This is useful if you want to execute one compute task from another synchronously avoiding deadlocks. To guarantee this, you need to make sure that a nested task is executed in a  thread pool different from the parent's tasks thread pool.

A custom pool is defined in `IgniteConfiguration` and has to have a unique name:
[block:code]
{
  "codes": [
    {
      "code": "IgniteConfiguration cfg = ...;\n\ncfg.setExecutorConfiguration(new ExecutorConfiguration(\"myPool\").setSize(16));",
      "language": "java"
    },
    {
      "code": "<bean id=\"grid.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"executorConfiguration\">\n    <list>\n      <bean class=\"org.apache.ignite.configuration.ExecutorConfiguration\">\n        <property name=\"name\" value=\"myPool\"/>\n        <property name=\"size\" value=\"16\"/>\n      </bean>\n    </list>\n  </property>\n  ...\n</bean>  ",
      "language": "xml"
    }
  ]
}
[/block]
Now, let's assume the Ignite Compute task below has to be executed by a Thread from `myPool` defined above:
[block:code]
{
  "codes": [
    {
      "code": "public class InnerRunnable implements IgniteRunnable {    \n    @Override public void run() {\n        System.out.println(\"Hello from inner runnable!\");\n    }\n}",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
To do that, you need to use `IgniteCompute.withExecutor()` executing the task right away or from an implementation of a parental task like it's shown below: 
[block:code]
{
  "codes": [
    {
      "code": "public class OuterRunnable implements IgniteRunnable {    \n    @IgniteInstanceResource\n    private Ignite ignite;\n    \n    @Override public void run() {\n        // Synchronously execute InnerRunnable in custom executor.\n        ignite.compute().withExecutor(\"myPool\").run(new InnerRunnable());\n    }\n}",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
The parental task's execution might be triggered this way and, in this scenario, it will be executed by the public pool size:
[block:code]
{
  "codes": [
    {
      "code": "ignite.compute().run(new OuterRunnable());",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "Undefined Thread Pool",
  "body": "If an Ignite Compute task is asked to be executed in a custom pool which is not defined on an Apache Ignite node, then a special warning message will be printed out to node's logs and the task will be picked up by the public pool for execution."
}
[/block]