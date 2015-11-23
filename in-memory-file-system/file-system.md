Ignite Hadoop Accelerator ships with Hadoop-compliant `IGFS File System` implementation called `IgniteHadoopFileSystem`. Hadoop can run over this file system in plug-n-play fashion and significantly reduce I/O and improve both, latency and throughput.  
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/cQXuII0VR8SApsUu1ndU",
        "ignite_filesystem.png",
        "1848",
        "1278",
        "#fa8c0c",
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
Apache Ignite Hadoop Accelerator performs file system operations within Ignite cluster. Several prerequisites must be satisfied.

1) `IGNITE_HOME` environment variable must be set and point to the root of Ignite installation directory.

2) Each cluster node must have Hadoop jars in CLASSPATH. 
See respective Ignite installation guide for your Hadoop distribution for details.

3) `IGFS` must be configured on the cluster node. See http://apacheignite.readme.io/v1.0/docs/igfs for details on how to do that.

4) To let `IGFS` accept requests from Hadoop, an endpoint should be configured (default configuration file is `${IGNITE_HOME}/config/default-config.xml`).
Ignite offers two endpoint types:
  * `shmem` - working over shared memory (not available on Windows);
  * `tcp` - working over standard socket API.
 
Shared memory endpoint is the recommended approach if the code executing file system operations is on the same machine as Ignite node. Note that `port` parameter is also used in case of shared memory communication mode to perform initial client-server handshake:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n  ...\n  <property name=\"ipcEndpointConfiguration\">\n    <bean class=\"org.apache.ignite.igfs.IgfsIpcEndpointConfiguration\">\n      <property name=\"type\" value=\"SHMEM\"/>\n      <property name=\"port\" value=\"12345\"/>\n    </bean>\n  </property>\n  ....\n</bean>",
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
      "code": "<bean class=\"org.apache.ignite.configuration.FileSystemConfiguration\">\n  ...\n  <property name=\"ipcEndpointConfiguration\">\n    <bean class=\"org.apache.ignite.igfs.IgfsIpcEndpointConfiguration\">\n      <property name=\"type\" value=\"TCP\"/>\n      <property name=\"host\" value=\"myHost\"/>\n      <property name=\"port\" value=\"12345\"/>\n    </bean>\n  </property>\n  ....\n</bean>",
      "language": "xml"
    },
    {
      "code": "FileSystemConfiguration fileSystemCfg = new FileSystemConfiguration();\n...\nIgfsIpcEndpointConfiguration endpointCfg = new IgfsIpcEndpointConfiguration();\nendpointCfg.setType(IgfsEndpointType.TCP);\nendpointCfg.setHost(\"myHost\");\n...\nfileSystemCfg.setIpcEndpointConfiguration(endpointCfg);",
      "language": "java"
    }
  ]
}
[/block]
If host is not set, it defaults to `127.0.0.1`.
If port is not set, it defaults to `10500`.

If ipcEndpointConfiguration is not set, then shared memory endpoint with default port will be used for Linux systems, and TCP endpoint with default port will be used for Windows.
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
To run Hadoop job using Ignite job tracker three prerequisites must be satisfied:

1) `IGNITE_HOME` environment variable must be set and point to the root of Ignite installation directory.

2) Hadoop must have Ignite JARS `${IGNITE_HOME}\libs\ignite-core-[version].jar` and `${IGNITE_HOME}\libs\hadoop\ignite-hadoop-[version].jar` in CLASSPATH. 

This can be achieved in several ways.
  * Add these JARs to `HADOOP_CLASSPATH` environment variable.
  * Copy or symlink these JARs to the folder where your Hadoop installation stores shared libraries.
See respective Ignite installation guide for your Hadoop distribution for details.

3) Ignite Hadoop Accelerator file system must be configured for the action your are going to perform.
At the very least you must provide fully qualified file system class name:
[block:code]
{
  "codes": [
    {
      "code": "<configuration>\n  ...\n  <property>\n    <name>fs.igfs.impl</name>\n    <value>org.apache.ignite.hadoop.fs.v1.IgniteHadoopFileSystem</value>\n  </property>\n  ...\n</configuration>",
      "language": "xml"
    }
  ]
}
[/block]
If you want to set Ignite File System as a default file system for your environment, then add the following property:
[block:code]
{
  "codes": [
    {
      "code": "<configuration>\n  ...\n  <property>\n    <name>fs.default.name</name>\n    <value>igfs:///</value>\n  </property>\n  ...\n</configuration>",
      "language": "xml"
    }
  ]
}
[/block]
Here value is an URL of endpoint of the Ignite node with IGFS. Rules how to this URL must look like provided at the end of this article.

There are several ways how to pass these configuration to your Hadoop jobs.
**First**, you may create separate `core-site.xml` file with these configuration properties and use it for job runs:
**Second**, you may override default `core-site.xml` of your Hadoop installation. This will force all Hadoop jobs to pick Ignite jobs tracker by default unless it is overriden on job level somehow. **Note that you will not be able to HDFS in this case.** 
**Third**, you may set these properties for particular job programmatically:
[block:code]
{
  "codes": [
    {
      "code": "Configuration conf = new Configuration();\n...\nconf.set(\"fs.igfs.impl\", \"org.apache.ignite.hadoop.fs.v1.IgniteHadoopFileSystem\");\nconf.set(\"fs.default.name\", \"igfs:///\");\n...\nJob job = new Job(conf, \"word count\");\n...",
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

If you created separate `core-site.xml`:
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
If you modified default `core-site.xml`, then `--config` option is not necessary:
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

[block:api-header]
{
  "type": "basic",
  "title": "File system URI"
}
[/block]
URI to access `IGFS` has the following structure: `igfs://[igfs_name@][host]:[port]/`, where:
  * `igfs_name` - optional name of IGFS to connect to (as specified in FileSystemConfiguration.setName(...)). Must always ends with `@` character. Defaults to `null` if omitted.
  * `host` - optional IGFS endpoint host (`IgfsIpcEndpointConfiguration.host`). Defaults to `127.0.0.1`.
  * `port` - optional IGFS endpoint port (`IgfsIpcEndpointConfiguration.port`). Defaults to `10500`.

Sample URIs:
  * `igfs://myIgfs@myHost:12345/` - connect to IGFS named `myIgfs` running on specific host and port;
  * `igfs://myIgfs@myHost/` - connect to IGFS named `myIgfs` running on specific host and default port;
  *  `igfs://myIgfs@/` - connect to IGFS named `myIgfs` running on localhost and default port;
  *  `igfs://myIgfs@:12345/` - connect to IGFS named `myIgfs` running on localhost and specific port;
  *  `igfs://myHost:12345/` - connect to IGFS with `null` name running on specific host and port;
  * `igfs://myHost/` - connect to IGFS with `null` name running on specific host and default port;
  * `igfs://:12345/` - connect to IGFS with `null` name running on localhost and specific port;
  * `igfs:///` - connect to IGFS with `null` name running on localhost and default port.