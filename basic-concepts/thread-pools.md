[block:api-header]
{
  "title": "Custom thread pools"
}
[/block]
It is possible to configure custom thread pools for compute tasks. This is useful if you want to synchronously execute one compute task from another without a risk of deadlock: just make sure that nested task is executed in another thread pool.

Custom pool should be defined in configuration before node start and must have unique name.
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
Compute task could be routed to particular pool using `IgniteCompute.withExecutor()` method.
[block:code]
{
  "codes": [
    {
      "code": "public class InnerRunnable implements IgniteRunnable {    \n    @Override public void run() {\n        System.out.println(\"Hello from inner runnable!\");\n    }\n}",
      "language": "java",
      "name": "InnerRunnable"
    }
  ]
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "public class OuterRunnable implements IgniteRunnable {    \n    @IgniteInstanceResource\n    private Ignite ignite;\n    \n    @Override public void run() {\n        // Execute nested runnable in custom executor.\n        ignite.compute().withExecutor(\"myPool\").run(new InnerRunnable());\n    }\n}",
      "language": "java",
      "name": "OuterRunnable"
    }
  ]
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "ignite.compute().run(new OuterRunnable());",
      "language": "java",
      "name": "Execute OuterRunnable"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "Undefined executor",
  "body": "If node doesn't have executor with particular name, a warning will be printed to log and compute task will be executed in public thread pool in the same way as regular compute task."
}
[/block]