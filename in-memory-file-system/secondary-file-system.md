Ignite Hadoop Accelerator contains implementation of `IGFS` secondary file system `IgniteHadoopIgfsSecondaryFileSystem` which allows read-through and write-through for any Hadoop `FileSystem` implementation.

To use the secondary file system specify it in `IGFS` configuration or in your Java source code:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n  ...\n  <property name=\"secondaryFileSystem\">\n    <bean class=\"org.apache.ignite.hadoop.fs.IgniteHadoopIgfsSecondaryFileSystem\">\n      <constructor-arg value=\"hdfs://myHdfs:9000\"/>                            \n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "FileSystemConfiguration fileSystemCfg = new FileSystemConfiguration();\n...\nIgniteHadoopIgfsSecondaryFileSystem hadoopFileSystem = new IgniteHadoopIgfsSecondaryFileSystem(\"hdfs://myHdfs:9000\");\n...\nfileSystemCfg.setSecondarFileSystem(hadoopFileSystem);",
      "language": "java"
    }
  ]
}
[/block]
Note that by default Apache Ignite will not have Hadoop libraries in the classpath during an Apache Ignite node startup. If you decide to use 'HDFS' as a secondary file system then you have to follow this steps in advance:

1. Use "Apache Ignite Hadoop Accelerator" edition of Ignite distribution (use -Dignite.edition=hadoop if you're building the distribution by yourself).
2. Set HADOOP_HOME environment variable before starting an Apache Ignite node if you're using Apache Hadoop distribution. If you use some other Hadoop distribution (HDP, Cloudera, BigTop, etc.) make sure that `/etc/default/hadoop` file exists and has appropriate content.

See respective Ignite installation guide for your Hadoop distribution for details.
  * [Installing on Apache Hadoop](doc:installing-on-apache-hadoop)
  * [Installing on Cloudera CDH](doc:installing-on-cloudera-cdh)
  * [Installing on Hortonworks HDP](doc:installing-on-hortonworks-hdp)


Alternatively, you can manually add necessary Hadoop dependencies to Ignite node classpath: these are dependencies of groupId "org.apache.hadoop" listed in file `modules/hadoop/pom.xml`. Currently they are the following:

hadoop-annotations
hadoop-auth
hadoop-common
hadoop-hdfs
hadoop-mapreduce-client-common
hadoop-mapreduce-client-core