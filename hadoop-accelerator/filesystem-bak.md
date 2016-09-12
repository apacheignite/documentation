Ignite Hadoop Accelerator ships with `FileSystem` implementation over `IGFS` called `IgniteHadoopFileSystem`. Thus, Hadoop can run over in-memory IGFS file system, what may significantly improve both latency and throughput.  
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/BRv2HAbiT76GsMkXo5N2_ignite_filesystem.png",
        "ignite_filesystem.png",
        "1848",
        "1278",
        "#f88c35",
        ""
      ]
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring Ignite node"
}
[/block]
IGFS is hosted on Ignite cluster. To let IGFS accept requests from Hadoop, you should configure an endpoint for communication (default configuration file is <IGNITE_HOME>/config/default-config.xml).
Ignite offers two endpoint types:
  * `shmem` - working over shared memory (not available on Windows);
  * `tcp` - working over standard socket API.
 
Shared memory endpoint is the recommended approach if the code executing file system operations is on the same machine as Ignite node. Note that `port` parameter is also used in case of shared memory communication mode to perform initial client-server handshake:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n  ...\n  <property name=\"ipcEndpointConfiguration\">\n    <bean class=\"org.apache.ignite.igfs.IgfsIpcEndpointConfiguration\">\n      <property name=\"type\" value=\"SHMEM\"/>\n      <property name=\"port\" value=\"10500\"/>\n    </map>\n  </property>\n  ....\n</bean>",
      "language": "xml"
    },
    {
      "code": "FileSystemConfiguration fileSystemCfg = new FileSystemConfiguration();\n...\nIgfsIpcEndpointConfiguration endpointCfg = new IgfsIpcEndpointConfiguration();\nendpointCfg.setType(IgfsEndpointType.SHMEM);\n...\nfileSystemCfg.setIpcEndpointConfiguration(endpointCfg);",
      "language": "java"
    }
  ]
}
[/block]
TCP endpoint should be used when Ignite node is either located on another machine, or shared memory is not available.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n  ...\n  <property name=\"ipcEndpointConfiguration\">\n    <bean class=\"org.apache.ignite.igfs.IgfsIpcEndpointConfiguration\">\n      <property name=\"type\" value=\"TCP\"/>\n      <property name=\"host\" value=\"myHost\"/>\n      <property name=\"port\" value=\"10500\"/>\n    </map>\n  </property>\n  ....\n</bean>",
      "language": "xml"
    },
    {
      "code": "FileSystemConfiguration fileSystemCfg = new FileSystemConfiguration();\n...\nIgfsIpcEndpointConfiguration endpointCfg = new IgfsIpcEndpointConfiguration();\nendpointCfg.setType(IgfsEndpointType.TCP);\nendpointCfg.setHost(\"myHost\");\n...\nfileSystemCfg.setIpcEndpointConfiguration(endpointCfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring Hadoop"
}
[/block]
In order to make Hadoop clients work correctly with IGFS the following 3 configuration items should be done:

1) Hadoop clients need `IGNITE_HOME` env variable to be set (this is needed for the Ignite libraries to find the logging configs).
    
2) Fully-qualified `IgniteHadoopFileSystem` class name should be provided to Hadoop configuration. (You can use template file `<IGNITE_HOME>/config/hadoop/core-site.ignite.xml` from Ignite installation). The IGFS port should be the same as the one specified in Ignite node configuration (`10500` in the example above):
[block:code]
{
  "codes": [
    {
      "code": "<configuration>\n  ...\n  <property>\n    <name>fs.default.name</name>\n    <value>igfs://igfs@myHost:10500</value>\n  </property>\n\n  <property>\n    <name>fs.igfs.impl</name>\n    <value>org.apache.ignite.hadoop.fs.v1.IgniteHadoopFileSystem</value>\n  </property>  \n  ...\n</configuration>",
      "language": "xml"
    }
  ]
}
[/block]
In order to use the custom `core-site.xml` you can either (a) create an alternative config directory, say, `cfg2`, put the custom configuration file there, and refer it either via `hadoop --config cfg2` client command line parameter, or via `HADOOP_CONF_DIR` environment variable when running the client, or, (b) you can rewrite global `core-site.xml` configuration file (which typically resides in `/etc/hadoop/conf/`, or `${HADOOP_HOME}/etc/hadoop/` directory in case of Apache distribution). However, in the latter case you will not be able to run ordinary HDFS as a secondary file system, use it if you plan to use IGFS only.

3) As in case of Ignite map-reduce framework, Hadoop clients need two Ignite libraries (`ignite-core-<IGNITE_VERSION>.jar`, `ignite-hadoop-<IGNITE_VERSION>.jar`) to be present in the Hadoop client classpath. You can either (a) add them via `HADOOP_CLASSPATH` variable, e.g.:
`export HADOOP_CLASSPATH=${IGNITE_HOME}/libs/ignite-core-<IGNITE_VERSION>.jar:${IGNITE_HOME}/libs/ignite-hadoop/ignite-hadoop-<IGNITE_VERSION>.jar`, or (b) copy or symlink these 2 libraries in Hadoop client library directory, e.g. `ln -s ${IGNITE_HOME}/libs/ignite-core-<IGNITE_VERSION>.jar /usr/lib/hadoop/client/`.
[block:api-header]
{
  "type": "basic",
  "title": "Accessing IgniteHadoopFileSystem"
}
[/block]
URI to access IGFS has the following structure: `igfs://[igfs_name@][host]:[port]/`, where:
  * `igfs_name` - optional name of IGFS to connect to (as specified in FileSystemConfiguration.setName(...)). Must always ends with `@` character. Default to `null` if omitted.
  * `host` - optional host where IGFS is running. Defaults to `127.0.0.1`.
  * `port` - optional port where IGFS is running. Defaults to `10500`.

Sample URIs:
  * `igfs://myIgfs@myHost:12345/` - connect to IGFS named `myIgfs` running on specific host and port;
  * `igfs://myIgfs@myHost/` - connect to IGFS named `myIgfs` running on specific host and default port;
  *  `igfs://myIgfs@/` - connect to IGFS named `myIgfs` running on local host and default port;
  *  `igfs://myIgfs@:12345/` - connect to IGFS named `myIgfs` running on local host and specific port;
  *  `igfs://myHost:12345/` - connect to IGFS with `null` name running on specific host and port;
  * `igfs://myHost/` - connect to IGFS with `null` name running on specific host and default port;
  * `igfs://:12345/` - connect to IGFS with `null` name running on local host and specific port;
  * `igfs:///` - connect to IGFS with `null` name running on local host and default port.