* [Overview](#overview)
* [Ignite YARN Application](#ignite-yarn-application)
* [Running Ignite YARN Application](#running-ignite-yarn-application)
* [Configuration](#configuration)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Integration with YARN supports scheduling and running Apache Ignite nodes in a YARN cluster.
YARN is a resource negotiator which provides a general runtime environment providing all the essentials to deploy, run and manage distributed applications. Its resource manager and isolation helps getting the most out of servers.
For information about YARN, refer to [http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) 
[block:api-header]
{
  "type": "basic",
  "title": "Ignite YARN Application"
}
[/block]
Deploying Apache Ignite cluster typically involves downloading the Apache Ignite distribution, changing configuration settings and starting the nodes up. Integration with YARN allows to avoid the actions. Apache Ignite Yarn Application allows to greatly simplify cluster deployment. The application consist from the following components: 

* Client downloads ignte distributive, puts necessary resources to HDFS, creates the necessary context for launching the task, launches the ApplicationMaster process.
* `Application master`. Once registration is successful the component will begin requesting of resource from Resource Manager to utilize resources for Apache Ignite nodes. `The Application Master` will maintain the Ignite cluster at desired total resources level (CPU, memory, etc).
* `Container` - the entity that runs Ignite Node on slaves.
[block:api-header]
{
  "type": "basic",
  "title": "Running Ignite YARN Application"
}
[/block]
For running Ignite Application requires YARN and Hadoop cluster are configured and running. For information on how to set up a the cluster please refer to http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html

1. Download Apache Ignite.
2. Configure properties file. Update any parameters which would like to change. See **Configuration** section below.
[block:code]
{
  "codes": [
    {
      "code": "# The number of nodes in the cluster.\nIGNITE_NODE_COUNT=2\n \n# The number of CPU Cores for each Apache Ignite node.\nIGNITE_RUN_CPU_PER_NODE=1\n \n# The number of Megabytes of RAM for each Apache Ignite node.\nIGNITE_MEMORY_PER_NODE=2048\n \n# The version ignite which will be run on nodes.\nIGNITE_VERSION=1.0.6",
      "language": "text"
    }
  ]
}
[/block]
3. Run the application.

`hadoop java jar ignite-yarn-<ignite-version>.jar ./ignite-yarn-<ignite-version>.jar cluster.properties`

4. In order to make sure that Application deployed correctly, do the following. Open YARN console at http://<hostname>:8088/cluster. If everything works OK then application with `Ignition` name.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/ISnxYZaaSyWC3Z6VP54H_AllApp.png",
        "AllApp.png",
        "1553",
        "371",
        "#6c9cb4",
        ""
      ]
    }
  ]
}
[/block]
5. Retrieve logs from the browser. To look through Ignite logs, click on `Logs` for any containers.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/y0U5GlGoTUeSl3qc8iMQ_ContainerLogs.png",
        "ContainerLogs.png",
        "1318",
        "258",
        "#425c67",
        ""
      ]
    }
  ]
}
[/block]
6. Click on `stdout` to get stdout logs and on `stderr` to get stderr logs.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/a3e8yeROWckIihHTWTwp_ContainerStdout.png",
        "ContainerStdout.png",
        "1384",
        "389",
        "#425c68",
        ""
      ]
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
All configurations are handled through environment variables or property file. Following configuration parameters can be optionally configured.
[block:parameters]
{
  "data": {
    "0-0": "IGNITE_XML_CONFIG",
    "h-0": "Name",
    "h-1": "Description",
    "h-2": "Default",
    "h-3": "Example",
    "0-1": "The hdfs path to Apache Ignite config file.",
    "0-2": "N/A",
    "0-3": "/opt/ignite/ignite-config.xml",
    "1-0": "IGNITE_WORK_DIR",
    "1-1": "The directory which will be used for saving Apache Ignite distributives.",
    "1-2": "./ignite-release",
    "1-3": "/opt/ignite/",
    "2-0": "IGNITE_RELEASES_DIR",
    "2-1": "The hdfs directory which will be used for saving Apache Ignite distributives.",
    "2-2": "/ignite/releases/",
    "2-3": "/ignite-rel/",
    "3-0": "IGNITE_USERS_LIBS",
    "3-1": "The hdfs path to libs which will be added to classpath.",
    "3-2": "N/A",
    "3-3": "/opt/libs/",
    "4-0": "IGNITE_MEMORY_PER_NODE",
    "4-1": "The number of Megabytes of RAM for each Apache Ignite node. This is the size of the Java heap.",
    "4-2": "2048",
    "4-3": "1024",
    "6-0": "IGNITE_HOSTNAME_CONSTRAINT",
    "6-1": "The constraint on slave hosts.",
    "6-2": "N/A",
    "6-3": "192.168.0.[1-100]",
    "7-0": "IGNITE_NODE_COUNT",
    "7-1": "The number of nodes in the cluster.",
    "7-2": "3",
    "7-3": "10",
    "8-0": "IGNITE_RUN_CPU_PER_NODE",
    "8-1": "The number of CPU Cores for each Apache Ignite node.",
    "8-2": "2",
    "8-3": "4",
    "9-0": "IGNITE_VERSION",
    "9-1": "The version ignite which will be run on nodes.",
    "9-2": "latest",
    "9-3": "1.0.5",
    "5-0": "IGNITE_MEMORY_OVERHEAD_PER_NODE",
    "5-1": "The amount of memory allocated to handle JVM native overheads, interned Strings,... but also off-heap memory if you use it. Memory requested to YARN for containers running Ignite node is the sum of  IGNITE_MEMORY_PER_NODE and IGNITE_MEMORY_OVERHEAD_PER_NODE.",
    "5-2": "IGNITE_MEMORY_PER_NODE * 0.10, with a minimum of 384",
    "5-3": "512",
    "10-3": "`/ignite/apache-ignite-fabric-1.7.0-bin.zip`",
    "10-2": "`N/A`",
    "10-0": "IGNITE_PATH",
    "10-1": "The hdfs path to Apache Ignite build. The property can be useful when yarn cluster running in net without access to internet."
  },
  "cols": 4,
  "rows": 11
}
[/block]