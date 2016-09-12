Ignite In-Memory MapReduce allows to effectively parallelize the processing data stored in any Hadoop file system. It eliminates the overhead associated with job tracker and task trackers in a standard Hadoop architecture while providing low-latency, HPC-style distributed processing.

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/sto4HGpGQxiYIGVFrAQX_ignite_mapreduce.png",
        "ignite_mapreduce.png",
        "800",
        "526",
        "#f09146",
        ""
      ]
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configure Ignite"
}
[/block]
Apache Ignite Hadoop Accelerator map-reduce engine processes Hadoop jobs within Ignite cluster. Several prerequisites must be satisfied.

1) `IGNITE_HOME` environment variable must be set and point to the root of Ignite installation directory.

2) Each cluster node must have Hadoop jars in CLASSPATH. 
See respective Ignite installation guide for your Hadoop distribution for details.

3) Cluster nodes accepts job execution requests listening particular socket. By default each Ignite node is listening for incoming requests on `127.0.0.1:11211`. You can override the host and port using `ConnectorConfiguration` class: 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"connectorConfiguration\">\n    <list>\n      <bean class=\"org.apache.ignite.configuration.ConnectorConfiguration\">\n        <property name=\"host\" value=\"myHost\" />\n        <property name=\"port\" value=\"12345\" />        \n    \t</bean>\n    </list>    \n  </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Run Ignite"
}
[/block]
When Ignite node is configured start it:
[block:code]
{
  "codes": [
    {
      "code": "$ bin/ignite.sh",
      "language": "shell"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configure Hadoop"
}
[/block]
To run Hadoop job using Ignite job tracker several prerequisites must be satisfied:

1) `IGNITE_HOME` environment variable must be set and point to the root of Ignite installation directory.

2) Hadoop must have Ignite JARS `${IGNITE_HOME}\libs\ignite-core-[version].jar` and `${IGNITE_HOME}\libs\hadoop\ignite-hadoop-[version].ja`" in CLASSPATH. 

This can be achieved in several ways.
  * Add these JARs to `HADOOP_CLASSPATH` environment variable.
  * Copy or symlink these JARs to the folder where your Hadoop installation stores shared libraries.
See respective Ignite installation guide for your Hadoop distribution for details.

3) Your Hadoop job must be configured to user Ignite job tracker. Two configuration properties are responsible for this:
  * `mapreduce.framework.name` must be set to `ignite`
  * `mapreduce.jobtracker.address` must be set to the host/port your Ignite nodes are listening.

This also can be achieved in several ways. **First**, you may create separate `mapred-site.xml` file with these configuration properties and use it for job runs:
[block:code]
{
  "codes": [
    {
      "code": "<configuration>\n  ...\n  <property>\n    <name>mapreduce.framework.name</name>\n    <value>ignite</value>\n  </property>\n  <property>\n    <name>mapreduce.jobtracker.address</name>\n    <value>127.0.0.1:11211</value>\n  </property>\n  ...\n</configuration>",
      "language": "xml"
    }
  ]
}
[/block]
**Second**, you may override default `mapred-site.xml` of your Hadoop installation. This will force all Hadoop jobs to pick Ignite jobs tracker by default unless it is overriden on job level somehow.

**Third**, you may set these properties for particular job programmatically:
[block:code]
{
  "codes": [
    {
      "code": "Configuration conf = new Configuration();\n...\nconf.set(MRConfig.FRAMEWORK_NAME,  IgniteHadoopClientProtocolProvider.FRAMEWORK_NAME);\nconf.set(MRConfig.MASTER_ADDRESS, \"127.0.0.1:11211);\n...\nJob job = new Job(conf, \"word count\");\n...",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Run Hadoop"
}
[/block]
How you run a job depends on how you have configured your Hadoop.

If you created separate `mapred-site.xml`:
[block:code]
{
  "codes": [
    {
      "code": "hadoop --config [path_to_config] [arguments]",
      "language": "shell"
    }
  ]
}
[/block]
If you modified default `mapred-site.xml`, then `--config` option is not necessary:
[block:code]
{
  "codes": [
    {
      "code": "hadoop [arguments]",
      "language": "shell"
    }
  ]
}
[/block]
If you start the job programmatically, then submit it:
[block:code]
{
  "codes": [
    {
      "code": "...\nJob job = new Job(conf, \"word count\");\n...\njob.submit();  ",
      "language": "java"
    }
  ]
}
[/block]