IGFS is able to work with another file systems through `IgfsSecondaryFileSystem` interface.
If particular file system path is configured to work in `DUAL_SYNC` or `DUAL_ASYNC` modes, IGFS will propagate all operations on this path or it's children to the secondary file system.

Ignite provides single secondary file system implementation over Hadoop 'FileSystem' called `IgniteHadoopIgfsSecondaryFileSystem`.

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