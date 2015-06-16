--------------
title: MapReduce bak
excerpt: High-performant Hadoop map-reduce engine.
--------------

Ignite In-Memory MapReduce allows to effectively parallelize the processing data stored in any Hadoop file system. It eliminates the overhead associated with job tracker and task trackers in a standard Hadoop architecture while providing low-latency, HPC-style distributed processing.

[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/aPKj2pqhRJuciy6AqFme",
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
  "title": "Configuring Ignite Node"
}
[/block]
Ignite Hadoop Accelerator map-reduce processing occurs on Ignite nodes. 
By default each node listen for incoming requests on the port 11211 (see `ConnectorConfiguration.DFLT_TCP_PORT`. 
[block:code]
{
  "codes": [
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n\nIgnite ignite = Ignition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
You can override the host and port using `ConnectorConfiguration` class (by default this is configured in IGNITE_HOME/config/default-config.xml):
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"connectorConfiguration\">\n    <list>\n      <bean class=\"org.apache.ignite.configuration.ConnectorConfiguration\">\n        <property name=\"host\" value=\"myHost\" />\n        <property name=\"port\" value=\"12345\" />        \n    \t</bean>\n    </list>    \n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n...\nConnectorConfiguration connCfg = new ConnectorConfiguration();\nconnCfg.setHost(\"myHost\");\nconnCfg.setPort(\"12345\");\ncfg.setConnectorConfiguration(connCfg);\n...\nIgnite ignite = Ignition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]
In order to serve as file system server and/or a job tracker Ignite node should also have some Hadoop libraries in the classpath. 
In case of **Cloudera** and **BigTop** distributions no action required since everything is configured automatically via `/etc/default/hadoop` configuration file. (Please do **not** export `HADOOP_HOME` explicitly in the Ignite node environment.)

In case of **Hortonworks** distribution create the file `/etc/default/hadoop` (of `0777` permission mode) with the following content:
[block:code]
{
  "codes": [
    {
      "code": "HDP=/usr/hdp/current\nexport HADOOP_HOME=$HDP/hadoop-client/\nexport HADOOP_COMMON_HOME=$HDP/hadoop-client/\nexport HADOOP_HDFS_HOME=$HDP/hadoop-hdfs-client/ \nexport HADOOP_MAPRED_HOME=$HDP/hadoop-mapreduce-client/",
      "language": "shell"
    }
  ]
}
[/block]
In contrast to the above recommendations, in case of **Apache** Hadoop distribution, export `HADOOP_HOME` to the value of the Hadoop installation root, e.g. export  `HADOOP_HOME=/opt/hadoop-2.6.0`.
[block:api-header]
{
  "type": "basic",
  "title": "Running Hadoop Jobs"
}
[/block]
To run Hadoop job over Ignite Hadoop Accelerator, you should configure map-reduce framework and Ignite node host/port either in Hadoop configuration files (`mapred-site.xml`) or programmatically. 

In order to make Hadoop client work correctly with Ignite map-reduce framework the following 3 configuration items should be done:

1) Hadoop clients need `IGNITE_HOME` env variable to be set (this is needed for the Ignite libraries to find the logging configs).
    
2) Hadoop clients need custom config `mapred-site.xml`: 
you can either (a) create an alternative config directory, say, `cfg2`, put the custom `mapred-site.xml` configuration file there, plus symlink or copy `core-site.xml` to the same directory,  and refer it either via `hadoop --config cfg2` command line parameter, or via `HADOOP_CONF_DIR` environment variable when running the client, or, (b) you can rewrite global `mapred-site.xml` configuration file (which typically resides in `/etc/hadoop/conf/`, or `${HADOOP_HOME}/etc/hadoop/` directory in case of Apache distribution). If you're planning to use Ignite map-reduce framework, you can stop yarn  resourcemanager and nodemanager processes because they will not be used.

Example of file `mapred-site.xml` (you can also use template file `<IGNITE_HOME>/config/hadoop/mapred-site.ignite.xml` provided with Ignite installation):
[block:code]
{
  "codes": [
    {
      "code": "<configuration>\n  ...\n  <property>\n    <name>mapreduce.framework.name</name>\n    <value>ignite</value>\n  </property>\n  <property>\n    <name>mapreduce.jobtracker.address</name>\n    <value>myHost:12345</value>\n  </property>\n  ...\n</configuration>",
      "language": "xml"
    }
  ]
}
[/block]
3) Hadoop clients need two Ignite libraries (`ignite-core-<IGNITE_VERSION>.jar`, `ignite-hadoop-<IGNITE_VERSION>.jar`) to be present in the Hadoop client classpath. You can either (a) add them via `HADOOP_CLASSPATH` variable, e.g.: 
`export HADOOP_CLASSPATH=${IGNITE_HOME}/libs/ignite-core-<IGNITE_VERSION>.jar:${IGNITE_HOME}/libs/ignite-hadoop/ignite-hadoop-<IGNITE_VERSION>.jar`, or (b) copy or symlink these 2 libraries in Hadoop client library directory, e.g. `ln -s ${IGNITE_HOME}/libs/ignite-core-<IGNITE_VERSION>.jar /usr/lib/hadoop/client/`.


Given the above 3 items, Hadoop client script `hadoop-ignited` can look like the following:
[block:code]
{
  "codes": [
    {
      "code": "IGNITE_VERSION=1.0.0-RC3\n\n# IGNITE_HOME is needed to resolve ignite logging configuration:\nexport IGNITE_HOME=/home/cloudera/ignite-hadoop-${IGNITE_VERSION}\n\n# The 2 Ignite jars need to be in the Hadoop classpath to load the Map-Reduce and file system implementations:\nexport HADOOP_CLASSPATH=${IGNITE_HOME}/libs/ignite-core-${IGNITE_VERSION}.jar:${IGNITE_HOME}/libs/ignite-hadoop/ignite-hadoop-${IGNITE_VERSION}.jar\n\nunset IGNITE_VERSION\n\n# Assume we have custom core-site.xml and mapred-site.xml configuration files into ${IGNITE_HOME}/cfg2/ directory:\nhadoop --config ${IGNITE_HOME}/cfg2 \"${@}\"",
      "language": "shell"
    }
  ]
}
[/block]

Now as both server and client sides are configured, you can also start the job programmically:
[block:code]
{
  "codes": [
    {
      "code": "Configuration conf = new Configuration();\n...\nJob job = new Job(conf, \"word count\");\n...\njob.submit();  ",
      "language": "java"
    }
  ]
}
[/block]