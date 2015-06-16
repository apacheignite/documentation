--------------
title: IGFS Modes
excerpt: Setup different operation modes for file system paths.
--------------

IGFS is able to operate in 4 modes: `PRIMARY`, `PROXY`, `DUAL_SYNC` and `DUAL_ASYNC`. Mode can be configured either for the whole file system or for particular paths. Modes are defined in `IgfsMode` enumeration. By default file system operates in `DUAL_ASYNC` mode. 

If secondary file system is not configured, all paths configured as `DUAL_SYNC` or `DUAL_ASYNC` will fallback to `PRIMARY` mode. 
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n  ...\n  <!-- Set default mode. -->\n  <property name=\"defaultMode\" value=\"DUAL_SYNC\" />     \n  <!-- Configure '/tmp' and all child paths to work in PRIMARY mode. -->\n  <property name=\"pathModes\">\n    <map>\n      <entry key=\"/tmp/.*\" value=\"PRIMARY\"/>      \n    </map>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "FileSystemConfiguration fileSystemCfg = new FileSystemConfiguration();\n...\nfileSystemCfg.setDefaultMode(IgfsMode.DUAL_SYNC);\n...\nMap<String, IgfsMode> pathModes = new HashMap<>();\npathModes.put(\"/tmp/.*\", IgfsMode.PRIMARY);\nfileSystemCfg.setPathModes(pathModes);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "PRIMARY mode"
}
[/block]
In this mode IGFS serves as a primary standalone distributed in-memory file system. Secondary file system is not used.
[block:api-header]
{
  "type": "basic",
  "title": "PROXY mode"
}
[/block]
IGFS is restricted to operate on paths in PROXY mode. Exception will be thrown if any operation is invoked on such path. 
[block:api-header]
{
  "type": "basic",
  "title": "DUAL_SYNC mode"
}
[/block]
In this mode IGFS will synchronously read-through from the secondary file system whenever data is requested and is not cached in memory, and synchronously write-through to it whenever data is updated/created in IGFS. Essentially, in this case IGFS serves as an intelligent caching layer on top of the secondary file system.
[block:api-header]
{
  "type": "basic",
  "title": "DUAL_ASYNC mode"
}
[/block]
Same as `DUAL_SYNC`, but seconadry file system reads and writes are performed asynchronously.
There is a lag between IGFS updates and secondary file system updates, however the performance of updates is significantly better than with `DUAL_SYNC` mode.