[block:api-header]
{
  "type": "basic",
  "title": "Shared Deployment"
}
[/block]
Shared deployment implies that Apache Ignite nodes are running independently from Apache Spark applications and store state even after Apache Spark jobs die. Similarly to Apache Spark there are three ways to deploy Apache Ignite to the cluster.

## Standalone deployment ##
In Standalone deployment mode Ignite nodes should be deployed together with Spark Worker nodes. Instruction on Ignite installation can be found [here](doc:getting-started). After you install Ignite on all worker nodes, start a node on each Spark worker with your config using `ignite.sh` script.

### Adding Ignite libraries to Spark classpath by default ###
Spark application deployment model allows dynamic jar distribution during application start. This model, however, has some drawbacks:
  *  Spark dynamic class loader does not implement `getResource` methods, so you will not be able to access resources located in jar files.
  * Java logger uses application class loader (not the context class loader) to load log handlers which results in `ClassNotFoundException` when using Java logging in Ignite.

There is a way to alter default Spark classpath for each launched application (this should be done on each machine of the Spark cluster, including master, worker and driver nodes).

1. Locate the `$SPARK_HOME/conf/spark-env.sh` file. If this file does not exist, create it from template using `$SPARK_HOME/conf/spark-env.sh.template`
2. Add the following lines to the end of the `spark-env.sh` file (uncomment the line setting `IGNITE_HOME` in case if you do not have it globally set):
[block:code]
{
  "codes": [
    {
      "code": "# Optionally set IGNITE_HOME here.\n# IGNITE_HOME=/path/to/ignite\n\nIGNITE_LIBS=\"${IGNITE_HOME}/libs/*\"\n\nfor file in ${IGNITE_HOME}/libs/*\ndo\n    if [ -d ${file} ] && [ \"${file}\" != \"${IGNITE_HOME}\"/libs/optional ]; then\n        IGNITE_LIBS=${IGNITE_LIBS}:${file}/*\n    fi\ndone\n\nexport SPARK_CLASSPATH=$IGNITE_LIBS",
      "language": "shell"
    }
  ]
}
[/block]
You can verify that the Spark classpath is changed by running `bin/spark-shell` and typing a simple import statement:
[block:code]
{
  "codes": [
    {
      "code": "scala> import org.apache.ignite.configuration._\nimport org.apache.ignite.configuration._",
      "language": "shell"
    }
  ]
}
[/block]
## Mesos deployment ##
Apache Ignite can be deployed on the Mesos cluster. Refer to the Mesos deployment instructions [here](doc:mesos-deployment).
[block:api-header]
{
  "type": "basic",
  "title": "Embedded Deployment"
}
[/block]
Embedded deployment means that Apache Ignite nodes are started inside Apache Spark job processes and are stopped when job dies. There is no need for additional deployment steps in this case. Apache Ignite code will be distributed to the worker machines using Apache Spark deployment mechanism and nodes will be started on all workers as a part of `IgniteContext` initialization.