* [Overview](#section-overview) 
* [System Pool](#section-system-pool)
* [Public Pool](#section-public-pool)
* [Queries Pool](#section-queries-pool)
* [Services Pool](#section-services-pool)
* [Striped Pool](#section-striped-pool)
* [Data Streamer Pool](#section-data-streamer-pool)
* [Custom Thread Pools](#section-custom-thread-pools)
[block:api-header]
{
  "title": "Overview"
}
[/block]
Apache Ignite creates and maintains a variety of Thread pools that are used for different purposes depending on an API that is being used. In this documentation, we list some of the most well-known internal pools and show how you can create a custom one. Refer to `IgniteConfiguration` javadoc to get the full list of all the pools available in Apache Ignite.
[block:api-header]
{
  "title": "System Pool"
}
[/block]
The system pool processes all the cache related operations except for SQL and some other types of queries that go to the [queries pool](#section-queries-pool). Also, this pool is responsible for Ignite Compute tasks' cancellation operations processing.

The default pool size is `2 x total number of cores`. Use `IgniteConfiguration.setSystemThreadPoolSize(...)` to change the pool size.
[block:api-header]
{
  "title": "Public Pool"
}
[/block]
The public pool is a work-horse of Apache Ignite compute grid. All computations get and processed by this pool.  

The default pool size is `2 x total number of cores`. Use `IgniteConfiguration.setPublicThreadPoolSize(...)` to change the pool size.
[block:api-header]
{
  "title": "Queries Pool"
}
[/block]
The queries pool takes care of all SQL, Scan and SPI queries that are being sent and executed across the cluster.

The default pool size is `2 x total number of cores`. Use `IgniteConfiguration.setQueryThreadPoolSize(...)` to change the pool size.
[block:api-header]
{
  "title": "Services Pool"
}
[/block]
Apache Ignite Service Grid calls go to the services' thread pool. Having dedicated pools for Ignite Service and Compute Grid components allows us to avoid threads starvation and deadlocks when a service implementation wants to call a computation or vice verse.

The default pool size is `2 x total number of cores`. Use `IgniteConfiguration.setServiceThreadPoolSize(...)` to change the pool size.
[block:api-header]
{
  "title": "Striped Pool"
}
[/block]
The striped pool helps to accelerate basic cache operations and transactions significantly by spreading the operations execution across multiple stripes that don't contend with each other.

The default pool size is `2 x total number of cores`. Use `IgniteConfiguration.setStripedPoolSize(...)` to change the pool size.
[block:api-header]
{
  "title": "Data Streamer Pool"
}
[/block]
The data streamer pool processes all the messages and requests coming from `IgniteDataStreamer` and a variety of streaming adapters that use `IgniteDataStreamer` internally. 

The default pool size is `2 x total number of cores`. Use `IgniteConfiguration.setDataStreamerThreadPoolSize(...)` to change the pool size.
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