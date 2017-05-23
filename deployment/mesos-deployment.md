* [Overview](#overview)
* [Ignite Mesos Framework](#ignite-mesos-framework)
* [Running Ignite Mesos Framework](#running-ignite-mesos-framework)
* [Configuration](#configuration)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite Framework supports scheduling and running Apache Ignite nodes in a Mesos cluster. 
Apache Mesos is a cluster manager which provides a general runtime environment providing all the essentials to deploy, run and manage distributed applications. Its resource management and isolation helps getting the most out of servers.
For information about Apache Mesos please refer to [http://mesos.apache.org/](http://mesos.apache.org/) 
[block:api-header]
{
  "type": "basic",
  "title": "Ignite Mesos Framework"
}
[/block]
Deploying Apache Ignite cluster typically involves downloading the Apache Ignite distribution, changing configuration settings and starting the nodes. Apache Ignite Mesos Framework consists of  `Scheduler` and `Task`, and allows to greatly simplify the cluster deployment.
* `Scheduler` registers itself at Mesos Master on scheduler startup. Once the registration is successful, the `Scheduler` will begin processing resource requests from Mesos Master to utilize resources for Apache Ignite nodes. The scheduler will maintain the Ignite cluster at the desired (and available) resource level (CPU, memory, etc.).
* `Task` launches an Ignite nodes on Mesos slave.
[block:api-header]
{
  "type": "basic",
  "title": "Running Ignite Mesos Framework"
}
[/block]
Running Ignite Mesos Framework requires Apache Mesos Cluster configured and running. For information on how to set up a Mesos cluster, please refer to https://docs.mesosphere.com/getting-started/datacenter/install/.
[block:callout]
{
  "type": "warning",
  "body": "Make sure that master and slave nodes listen on correct ip addresses. Otherwise, there is no guarantee that Mesos Cluster will correctly work."
}
[/block]
## **Run the Framework via Marathon** 

Currently, the recommended way to run the framework is to run it via Marathon.

1. Install marathon. See https://docs.mesosphere.com/getting-started/datacenter/install/ marathon section.
2. Download Apache Ignite and upload `libs\optional\ignite-mesos\ignite-mesos-<ignite-version>-jar-with-dependencies.jar` file to any cloud storage. (for example Amazon S3 storage and etc.).
3. Copy the following application definition (in JSON format) and save to `marathon.json` file. Update any parameters which would like to change. 
[block:code]
{
  "codes": [
    {
      "code": "{\n  \"id\": \"ignition\",\n  \"instances\": 1,\n  \"cpus\": 2,\n  \"mem\": 2048,\n  \"ports\": [0],\n  \"uris\": [\n    \"http://host/ignite-mesos-<ignite-version>-jar-with-dependencies.jar\"\n  ],\n  \"env\": {\n    \"IGNITE_NODE_COUNT\": \"4\",\n    \"MESOS_MASTER_URL\": \"zk://localhost:2181/mesos\",\n    \"IGNITE_RUN_CPU_PER_NODE\": \"2\",\n    \"IGNITE_MEMORY_PER_NODE\": \"2048\",\n    \"IGNITE_VERSION\": \"1.0.5\",\n    \"MESOS_USER\" : \"userAAAAA\",\n    \"MESOS_ROLE\" :  \"role1\"\n  },\n  \"cmd\": \"java -jar ignite-mesos-<ignite-version>-jar-with-dependencies.jar\"\n}",
      "language": "json"
    }
  ]
}
[/block]
A role name must be a valid directory name, so it cannot:
  • Be an empty string
  • Be . or ..
  • Start with -
  • Contain any slash, backspace, or whitespace character

If there are no restrictions set on the cluster, the framework will try to occupy all the resources in Mesos cluster. See [Configuration](doc:mesos-deployment#section-configuration) section below.

4. Send POST request with the application definition to Marathon, using CURL or other tools. 
[block:code]
{
  "codes": [
    {
      "code": "curl -X POST -H \"Content-type: application/json\" --data-binary @marathon.json http://<marathon-ip>:8080/v2/apps/",
      "language": "text"
    }
  ]
}
[/block]
5. In order to make sure that Apache Mesos Framework is deployed correctly, do the following - Open Marathon UI  at `http://<marathon-ip>:8080`. Make sure that an application with the name `ignition` exists and its status is `Running`.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/sTIbAfcdScKoDCSAJ6Q5_marathon.png",
        "marathon.png",
        "1087",
        "639",
        "#16b891",
        ""
      ]
    }
  ]
}
[/block]
6. Open Mesos console at `http://<master-ip>:5050`. If everything works fine, then tasks with name like `Ignite node N` should have the state `RUNNING`. In this example, N is equal to 4. See example `marathon.json` file - "IGNITE_NODE_COUNT": "4"
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/WSZ5mvnqQzy0dsUeq9WQ_mesos.png",
        "mesos.png",
        "1260",
        "861",
        "#99573c",
        ""
      ]
    }
  ]
}
[/block]
7. Mesos allows to retrieve tasks' logs from the browser. To look through Ignite logs, click on `Sandbox` in the Active Tasks table.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/qUqG485tRtKS50JCp7yn_mesos_sandbox.png",
        "mesos_sandbox.png",
        "1225",
        "642",
        "#97aac0",
        ""
      ]
    }
  ]
}
[/block]
8. Click on `stdout` to get stdout logs and on `stderr` to get stderr logs.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/Ch5VkVm1Q5qGvrGEYF4k_mesos_sandbox_stdout.png",
        "mesos_sandbox_stdout.png",
        "1493",
        "853",
        "#182b54",
        ""
      ]
    }
  ]
}
[/block]
## **Run the Framework via JAR file**
1. Download Ignite package and go to `libs\optional\ignite-mesos\` folder.
2. Run the framework using the following command
[block:code]
{
  "codes": [
    {
      "code": "java -jar ignite-mesos-<ignite-version>-jar-with-dependencies.jar",
      "language": "text"
    }
  ]
}
[/block]
or
[block:code]
{
  "codes": [
    {
      "code": "java -jar ignite-mesos-<ignite-version>-jar-with-dependencies.jar properties.prop",
      "language": "text"
    }
  ]
}
[/block]
where `properties.prop` is a property file. If the file is not provided, the framework will try to occupy all resources in the Mesos cluster. Here is an example property file:
[block:code]
{
  "codes": [
    {
      "code": "# The number of nodes in the cluster.\nIGNITE_NODE_COUNT=1\n\n# Mesos ZooKeeper URL to locate leading master.\nMESOS_MASTER_URL=zk://localhost:2181/mesos\n\n# The number of CPU Cores for each Apache Ignite node.\nIGNITE_RUN_CPU_PER_NODE=4\n\n# The number of Megabytes of RAM for each Apache Ignite node.\nIGNITE_MEMORY_PER_NODE=4096\n\n# The version ignite which will be run on nodes.\nIGNITE_VERSION=1.0.5",
      "language": "text"
    }
  ]
}
[/block]
3. In order to make sure that Apache Mesos Framework is deployed correctly, do the following - Open Mesos console at `http://<master-ip>:5050`. If everything works fine, the tasks with name like `Ignite node N` should have the state `RUNNING`. In this example, N is equal to 1. See example `properties.prop` file - "IGNITE_NODE_COUNT": "1"
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/eEmfch9cQcSQiM27gSqT_Mesos_console.png",
        "Mesos_console.png",
        "1385",
        "592",
        "#5598ce",
        ""
      ]
    }
  ]
}
[/block]
4. Mesos allows to retrieve tasks' logs from browser. To look through Ignite logs, click on `Sandbox`.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/s6fvlxNcQz66nhq8e9zH_Sandbox.png",
        "Sandbox.png",
        "1384",
        "395",
        "#5b9bd0",
        ""
      ]
    }
  ]
}
[/block]
5. Click on `stdout` to get stdout logs and on `stderr` to get stderr logs.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/MEksCWXBTRq5WpUyp4CJ_stdout.png",
        "stdout.png",
        "943",
        "574",
        "#cc8c38",
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
All configurations are handled through environment variables (this lends itself well to being easy to configure marathon to run the framework) or property file. Following configuration parameters can be optionally configured.
[block:parameters]
{
  "data": {
    "h-0": "Name",
    "h-1": "Description",
    "h-2": "Default",
    "0-0": "`IGNITE_RUN_CPU_PER_NODE`",
    "0-1": "The number of CPU Cores for each Apache Ignite node.",
    "0-2": "`UNLIMITED`",
    "h-3": "Example",
    "0-3": "`2`",
    "1-0": "`IGNITE_MEMORY_PER_NODE`",
    "1-1": "The number of Megabytes of RAM for each Apache Ignite node.",
    "1-2": "`UNLIMITED`",
    "1-3": "`1024`",
    "2-0": "`IGNITE_DISK_SPACE_PER_NODE`",
    "2-1": "The number of Megabytes of Disk for each Apache Ignite node.",
    "2-2": "`1024`",
    "2-3": "`2048`",
    "3-0": "`IGNITE_NODE_COUNT`",
    "3-1": "The number of nodes in the cluster.",
    "3-2": "`5`",
    "3-3": "`10`",
    "4-0": "`IGNITE_TOTAL_CPU`",
    "4-1": "The number of CPU Cores for Ignite cluster.",
    "4-2": "`UNLIMITED`",
    "4-3": "`5`",
    "5-0": "`IGNITE_TOTAL_MEMORY`",
    "5-1": "The number of Megabytes of RAM for Ignite cluster.",
    "5-2": "`UNLIMITED`",
    "5-3": "`16384`",
    "6-0": "`IGNITE_TOTAL_DISK_SPACE`",
    "6-1": "The number of Megabytes of Disk for each Apache Ignite cluster.",
    "6-2": "`UNLIMITED`",
    "6-3": "`5120`",
    "7-0": "`IGNITE_MIN_CPU_PER_NODE`",
    "7-1": "The minimum number of CPU cores required to run Apache Ignite node.",
    "7-2": "`1`",
    "7-3": "`4`",
    "8-0": "`IGNITE_MIN_MEMORY_PER_NODE`",
    "8-1": "The minimum number of Megabytes of RAM cores required to run Apache Ignite node.",
    "8-2": "`256`",
    "8-3": "`1024`",
    "9-0": "`IGNITE_VERSION`",
    "9-1": "The version ignite which will be run on nodes.",
    "9-2": "`latest`",
    "9-3": "`1.0.5`",
    "10-0": "`IGNITE_WORK_DIR`",
    "10-1": "The directory which will be used for saving Apache Ignite distributives.",
    "10-2": "`ignite-release`",
    "10-3": "`/opt/ignite/`",
    "11-0": "`IGNITE_XML_CONFIG`",
    "11-1": "The path to Apache Ignite config file.",
    "11-2": "`N/A`",
    "11-3": "`/opt/ignite/ignite-config.xml`",
    "13-0": "`IGNITE_USERS_LIBS`",
    "13-1": "The path to libs which will be added to classpath.",
    "13-2": "`N/A`",
    "13-3": "`/opt/libs/`",
    "15-0": "`MESOS_MASTER_URL`",
    "15-1": "Mesos ZooKeeper URL to locate leading master.",
    "15-2": "`zk://localhost:2181/mesos`",
    "15-3": "`zk://176.0.1.45:2181/mesos`\nor\n`176.0.1.45:2181`",
    "12-0": "`IGNITE_CONFIG_XML_URL`",
    "12-1": "The url to Apache Ingite config file.",
    "12-2": "`N/A`",
    "12-3": "`https://example.com/default-config.xml`",
    "14-0": "`IGNITE_USERS_LIBS_URL`",
    "14-1": "Comma separated list of ulrs to libs which will be added to classpath.",
    "14-2": "`N/A`",
    "14-3": "`https://example.com/lib.zip,`\n`https://example.com/lib1.zip`",
    "17-0": "`IGNITE_PACKAGE_PATH`",
    "17-1": "Path to Apache Ignite archive. This parameter may be useful when access to internet is limited.",
    "17-2": "`N/A`",
    "17-3": "`/opt/ignite/apache-ignite-fabric-1.5.0.final-bin.zip`",
    "16-0": "`IGNITE_PACKAGE_URL`",
    "16-1": "URL to Apache Ignite archive. This parameter could be used instead of the IGNITE_VERSION parameter.",
    "16-2": "`N/A`",
    "16-3": "`http://apache-mirror.rbc.ru/pub/apache//ignite/1.7.0/apache-ignite-1.7.0-src.zip`",
    "18-0": "`IGNITE_HTTP_SERVER_IDLE_TIMEOUT`",
    "18-1": "Sets the maximum Idle time (milliseconds) for a http connection, which will be used for jetty  server. The server provides resources for ignite mesos framework such as ignite archive, user's libs, configurations and etc.",
    "18-2": "`30000`",
    "18-3": "`300000`"
  },
  "cols": 4,
  "rows": 19
}
[/block]