Ignite Hadoop Accelerator contains implementation of `IGFS` secondary file system `IgniteHadoopIgfsSecondaryFileSystem` which allows read-through and write-through for any Hadoop `FileSystem` implementation.
To use secondary file system set it in `IGFS` configuration:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\" parent=\"igfsCfgBase\">\n  <property name=\"secondaryFileSystem\">\n    <bean class=\"org.apache.ignite.hadoop.fs.IgniteHadoopIgfsSecondaryFileSystem\">\n      <constructor-arg name=\"uri\" value=\"hdfs://myHost:9000\"/>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "FileSystemConfiguration fileSystemCfg = new FileSystemConfiguration();\n...\nIgfsSecondaryFileSystem secondaryFileSystem = new IgniteHadoopIgfsSecondaryFileSystem(\"hdfs://myHdfs:9000\");\nfileSystemCfg.setSecondaryFileSystem(secondaryFileSystem);\n...",
      "language": "java"
    }
  ]
}
[/block]
In addition to secondary file system URI you may also optionally specify path to it's configuration file and user name.