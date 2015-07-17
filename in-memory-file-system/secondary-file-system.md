Ignite Hadoop Accelerator contains implementation of `IGFS` secondary file system `IgniteHadoopIgfsSecondaryFileSystem` which allows read-through and write-through for any Hadoop `FileSystem` implementation.
To use secondary file system set it in `IGFS` configuration:
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