* [Overview](#overview)
* [Round-Robin Load Balancing](#round-robin-load-balancing)
  * [Per-Task Mode](#section-per-task-mode)
  * [Global Mode](#section-global-mode)
* [Random and Weighted Load Balancing](#random-and-weighted-load-balancing)
*[Job Stealing](#job-stealing)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Load balancing component balances job distribution among cluster nodes. In Apache Ignite, load balancing is achieved via `LoadBalancingSpi` which controls load on all nodes and makes sure that every node in the cluster is equally loaded. In homogeneous environments with homogeneous tasks, load balancing is achieved by random or round-robin policies. However, in many other use cases, especially under uneven load, more complex adaptive load-balancing policies may be needed.
[block:callout]
{
  "type": "info",
  "body": "Note that load balancing is triggered whenever your jobs are not collocated with data or have no real preference on which node to execute. If [Collocation Of Compute and Data](doc:collocate-compute-and-data) is used, then data affinity takes priority over load balancing.",
  "title": "Data Affinity"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Round-Robin Load Balancing"
}
[/block]
`RoundRobinLoadBalancingSpi` iterates through nodes in a round-robin fashion and picks the next sequential node. Two modes of operation are supported: per-task and global. Global mode is used by default.

##Per-Task Mode
When configured in per-task mode, implementation will pick a random node at the beginning of every task execution and then sequentially iterate through all nodes in topology starting from the picked node. For cases when split size is equal to the number of nodes, this mode guarantees that all nodes will participate in the split.

##Global Mode
When configured in global mode, a single sequential queue of nodes is maintained for all tasks and the next node in the queue is picked every time. In this mode (unlike in per-task mode) it is possible that even if split size may be equal to the number of nodes, some jobs within the same task will be assigned to the same node whenever multiple tasks are executing concurrently.
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"grid.custom.cfg\" class=\"org.apache.ignite.IgniteConfiguration\" singleton=\"true\">\n  ...\n  <property name=\"loadBalancingSpi\">\n    <bean class=\"org.apache.ignite.spi.loadbalancing.roundrobin.RoundRobinLoadBalancingSpi\">\n      <!-- Set to per-task round-robin mode (this is default behavior). -->\n      <property name=\"perTask\" value=\"true\"/>\n    </bean>\n  </property>\n  ...\n</bean>",
      "language": "xml",
      "name": null
    },
    {
      "code": "RoundRobinLoadBalancingSpi = new RoundRobinLoadBalancingSpi();\n \n// Configure SPI to use per-task mode (this is default behavior).\nspi.setPerTask(true);\n \nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default load balancing SPI.\ncfg.setLoadBalancingSpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Random and Weighted Load Balancing"
}
[/block]
`WeightedRandomLoadBalancingSpi` picks a random node for job execution by default. You can also optionally assign weights to nodes, so nodes with larger weights will end up getting proportionally more jobs routed to them. By default all nodes get equal weight of 10.
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"grid.custom.cfg\" class=\"org.apache.ignite.IgniteConfiguration\" singleton=\"true\">\n  ...\n  <property name=\"loadBalancingSpi\">\n    <bean class=\"org.apache.ignite.spi.loadbalancing.weightedrandom.WeightedRandomLoadBalancingSpi\">\n      <property name=\"useWeights\" value=\"true\"/>\n      <property name=\"nodeWeight\" value=\"10\"/>\n    </bean>\n  </property>\n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "WeightedRandomLoadBalancingSpi = new WeightedRandomLoadBalancingSpi();\n \n// Configure SPI to used weighted random load balancing.\nspi.setUseWeights(true);\n \n// Set weight for the local node.\nspi.setWeight(10);\n \nIgniteConfiguration cfg = new IgniteConfiguration();\n \n// Override default load balancing SPI.\ncfg.setLoadBalancingSpi(spi);\n \n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "Job Stealing"
}
[/block]
Quite often grids are deployed across many computers some of which may be more powerful than others. Enabling `JobStealingCollisionSpi` helps to avoid jobs being stuck at a slower node, as they will be stolen by a faster node.

`JobStealingCollisionSpi` supports job stealing from over-utilized nodes to under-utilized nodes. This SPI is especially useful if you have some jobs that complete fast, while others are sitting in the waiting queue on slower nodes. In such a case, the waiting jobs will be stolen from the slower node and moved to the fast under-utilized node.

Here is an example of how to configure `JobStealingCollisionSpi`:
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"grid.custom.cfg\" class=\"org.apache.ignite.IgniteConfiguration\" singleton=\"true\">\n  ...\n  <property name=\"failoverSpi\">\n     <bean class=\"org.apache.ignite.spi.failover.jobstealing.JobStealingFailoverSpi\"/>\n \t</property>\n  <property name=\"collisionSpi\">\n    <bean class=\"org.apache.ignite.spi.collision.jobstealing.JobStealingCollisionSpi\">\n      <property name=\"activeJobsThreshold\" value=\"50\"/>\n      <property name=\"waitJobsThreshold\" value=\"0\"/>\n      <property name=\"messageExpireTime\" value=\"1000\"/>\n      <property name=\"maximumStealingAttempts\" value=\"10\"/>\n      <property name=\"stealingEnabled\" value=\"true\"/>\n      <property name=\"stealingAttributes\">\n        <map>\n            <entry key=\"node.segment\" value=\"foobar\"/>\n        </map>\n      </property>\n    </bean>\n  </property>\n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "JobStealingCollisionSpi spi = new JobStealingCollisionSpi();\n\n // Configure number of waiting jobs\n // in the queue for job stealing.\n spi.setWaitJobsThreshold(10);\n\n // Configure message expire time (in milliseconds).\n spi.setMessageExpireTime(1000);\n\n // Configure stealing attempts number.\n spi.setMaximumStealingAttempts(10);\n\n // Configure number of active jobs that are allowed to execute\n // in parallel. This number should usually be equal to the number\n // of threads in the pool (default is 100).\n spi.setActiveJobsThreshold(50);\n\n // Enable stealing.\n spi.setStealingEnabled(true);\n\n // Set stealing attribute to steal from/to nodes that have it.\n spi.setStealingAttributes(Collections.singletonMap(\"node.segment\", \"foobar\"));\n \n // Enable `JobStealingFailoverSpi`\n JobStealingFailoverSpi failoverSpi = new JobStealingFailoverSpi();\n\n IgniteConfiguration cfg = new IgniteConfiguration();\n\n // Override default Collision SPI.\n cfg.setCollisionSpi(spi);\n \n cfg.setFailoverSpi(failoverSpi);",
      "language": "text"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Note that this SPI must always be used in conjunction with `JobStealingFailoverSpi`. Also note that job metrics update should be enabled (i.e. IgniteConfiguration.getMetricsUpdateFrequency() should be set to a positive value) in order for this SPI to work properly."
}
[/block]