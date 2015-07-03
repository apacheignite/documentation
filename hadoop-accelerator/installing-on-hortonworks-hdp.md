This article explains how to install Apache Ignite Hadoop Accelerator on Hortonworks HDP distribution.

Please read the following articles first to get better understanding of product's architecture:
* http://apacheignite.readme.io/v1.0/docs/overview
* http://apacheignite.readme.io/v1.0/docs/map-reduce
* http://apacheignite.readme.io/v1.0/docs/file-system
[block:api-header]
{
  "type": "basic",
  "title": "Ignite"
}
[/block]
1) Download the latest version of Apache Ignite Hadoop Accelerator and unpack it somewhere.

2) Set `IGNITE_HOME` variable to the directory where you unpacked Apache Ignite Hadoop Accelerator.

3) To let Ignite find required HDP JARs, create a file `/etc/default/hadoop` with the following content:
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
4) If you are going to use Ignite `FileSystem` implementation, configure `IGFS` in XML configuration (default configuration is `${IGNITE_HOME}/config/default-config.xml`):
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"fileSystemConfiguration\">\n    <list>\n      <bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n        <property name=\"metaCacheName\" value=\"myMetaCache\" />\n        <property name=\"dataCacheName\" value=\"myDataCache\" />       \n      </bean>\n    </list>    \n  </property>\n  ...\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
IGFS with this configuration will listen for incoming file system requests with default endpoint bound to `127.0.0.1:10500`. 
If you want to override it, provide alternate `ipcEndpointConfiguration`  (see http://apacheignite.readme.io/v1.0/docs/file-system).

5) If you are going to use Ignite map-reduce engine for your jobs, no additional configuration is required, as node will listen for job execution requests with default endpoint bound to `127.0.0.1:11211`. 
If you want to override it, provide alternate 'ConnectorConfiguration' (see http://apacheignite.readme.io/v1.0/docs/map-reduce).

At this point Ignite node is ready to be started:
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
  "title": "HDP"
}
[/block]
1) Enusre that `IGNITE_HOME` environment variable is set and points to the directory where you unpacked Apache Ignite Hadoop Accelerator.

2) Go to the **Ambari** web application and stop the following services: HDFS, MapReduce2 and YARN.

3) Set Ignite JARs `${IGNITE_HOME}\libs\ignite-core-[version].jar` and `${GNITE_HOME}\libs\hadoop\ignite-hadoop-[version].ja`" to HDP CLASSPATH. 
To do this go to **HDFS** -> **Configs** -> **Advanced hadoop-env** section and edit the property **hadoop-env template**. Find the first export of `HADOOP_CLASSPATH` which typically looks as follows:
[block:code]
{
  "codes": [
    {
      "code": "export HADOOP_CLASSPATH=${HADOOP_CLASSPATH}${JAVA_JDBC_LIBS}:${MAPREDUCE_LIBS}",
      "language": "shell"
    }
  ]
}
[/block]
Place the following after this line:
[block:code]
{
  "codes": [
    {
      "code": "export HADOOP_CLASSPATH=${HADOOP_CLASSPATH}:${IGNITE_HOME}/libs/ignite-core-1.0.0.jar:${IGNITE_HOME}/libs/hadoop/ignite-hadoop-1.0.0.jar",
      "language": "shell"
    }
  ]
}
[/block]
4) If you want to use Ignite `FileSystem`, configure it in a separate `core-site.xml` file:
[block:code]
{
  "codes": [
    {
      "code": "<configuration>\n  ...\n  <property>\n    <name>fs.default.name</name>\n    <value>igfs:///</value>\n  </property>\n  ...\n  <property>\n    <name>fs.igfs.impl</name>\n    <value>org.apache.ignite.hadoop.fs.v1.IgniteHadoopFileSystem</value>\n  </property>  \n  ...\n</configuration>",
      "language": "xml"
    }
  ]
}
[/block]
Alternatively you can define `fs.igfs.impl` property in default `core-site.xml`: go to **HDFS** -> **Configs** -> **Custom core-site** and add the property `fs.igfs.impl` with value `org.apache.ignite.hadoop.fs.v1.IgniteHadoopFileSystem`.

Also, you can define fs.default.name property in default `core-site.xml`: go to **HDFS** -> **Configs** -> **Advanced core-site** and add the property `fs.default.name` with value `igfs:///`. 
Note that if you change `fs.default.name` to use Ignite FileSystem in default `core-site.xml`, HDP will not be able to work with `HDFS` anymore. If you want to use both Ignite `FileSystem` and `HDFS` at the same time, consider creating separate configuration file.

5) If you want to use Ignite `MapReduce` job tracker, configure it in a separate `mapred-site.xml` file:
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
Alternatively you can configure these properties in default HDP `mapred-site.xml`: 
* Create an empty archive anywhere in your system (e.g. with name `dummy.tar.gz`).
* Go to **MapReduce2** -> **Configs** -> **Advanced mapred-site** section and change the following properties:
`mapreduce.framework.name` = `ignite`
`mapreduce.application.framework.path` = `/path/to/dummy.tar.gz`
The latter property is required because HDP requires to define archive with Hadoop framework but in case of Ignite it doesn't make sense.
* Go to **MapReduce2** -> **Configs** -> **Custom mapred-site** section and change the following property:
`mapreduce.jobtracker.address` = `127.0.0.1:11211`.

6) If you made any changes to default configuration(s), save them and restart all HDP services.

7) At this point installation is finished and you can start running jobs. 
Run a job with separate `core-site.xml` and/or `mapred-site.xml` configuration files:
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
Run a job with default configuration:
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