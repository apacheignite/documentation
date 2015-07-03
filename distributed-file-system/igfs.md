Ignite File System (IGFS) is an in-memory file system allowing work with files and directories over existing cache infrastructure. 
IGFS can either work as purely in-memory file system, or delegate to another file system (e.g. various Hadoop file system implementations) acting as a caching layer.
In addition IGFS provides API to execute map-reduce tasks over file system data.
[block:api-header]
{
  "type": "basic",
  "title": "IgniteFileSystem"
}
[/block]
`IgniteFileSystem` interface is a gateway into Ignite file system implementation. It provides methods for regular file system operations such as `create`, `delete`, `mkdirs`, etc., as well as methods for map-reduce tasks execution.
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\n// Obtain instance of IGFS named \"myFileSystem\".\nIgniteFileSystem fs = ignite.fileSystem(\"myFileSystem\");",
      "language": "java"
    }
  ]
}
[/block]
IGFS can be configured either through Spring XML file or programmatically.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  ...\n  <property name=\"fileSystemConfiguration\">\n    <list>\n      <bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n        <!-- Distinguished file system name. -->\n      \t<property name=\"name\" value=\"myFileSystem\" />\n        <!-- Name of the cache where file system structure will be stored. Should be configured separately. -->\n      \t<property name=\"metaCacheName\" value=\"myMetaCache\" />\n        <!-- Name of the cache where file data will be stored. Should be configured separately. -->\n        <property name=\"dataCacheName\" value=\"myDataCache\" />      \t\n    \t</bean>\n    </list>    \n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n...\nFileSystemConfiguration fileSystemCfg = new FileSystemConfiguration();\nfileSystemCfg.setName(\"myFileSystem\");\nfileSystemCfg.setMetaCacheName(\"myMetaCache\");\nfileSystemCfg.setDataCacheName(\"myDataCache\");\n...\ncfg.setFileSystemConfiguration(fileSystemCfg);\n",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Work with files and directories"
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "IgniteFileSystem fs = ignite.fileSystem(\"myFileSystem\");\n\n// Create directory.\nIgfsPath dir = new IgfsPath(\"/myDir\");\n\nfs.mkdirs(dir);\n\n// Create file and write some data to it.\nIgfsPath file = new IgfsPath(dir, \"myFile\");\n\ntry (OutputStream out = fs.create(file, true)) {\n    out.write(...);  \n}\n\n// Read from file.\ntry (InputStream in = fs.open(file)) {\n    in.read(...);\n}\n\n// Delete directory.\nfs.delete(dir, true);",
      "language": "java"
    }
  ]
}
[/block]