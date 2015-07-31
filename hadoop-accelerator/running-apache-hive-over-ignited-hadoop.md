This article explains how to properly configure and start Hive over Hadoop accelerated by Ignite. It also shows how to start HiveServer2 and a remote client with such configuration.
[block:api-header]
{
  "type": "basic",
  "title": "Prerequisites"
}
[/block]
We assume that Hadoop is already installed and configured to run over Ignite, and Ignite node(s) providing `IGFS` file system and map-reduce job tracker functionality is up and running.

You will also need to install Hive: http://hive.apache.org/.
[block:api-header]
{
  "type": "basic",
  "title": "Starting Hive"
}
[/block]
Here are the steps required to run Hive over "Ignited" Hadoop:
* Provide the location of correct `hadoop` executable. This can be done either with adding path to the executable file into `PATH` environment variable (note that this executable should be located in a folder named `bin/` anyway), or by specifying `HADOOP_HOME` environment variable.
* Provide the location of configuration files (`core-site.xml`, `hive-site.xml`, `mapred-site.xml`). To do this put all these files in a directory and specify the path to this directory as `HIVE_CONF_DIR` environment variable.
[block:callout]
{
  "type": "success",
  "body": "We recommend to use Hive template configuration file `<IGNITE_HOME>/config/hadoop/hive-site.ignite.xml` to get Ignite specific settings.",
  "title": "Configuration Template"
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "There is a potential [issue](http://stackoverflow.com/questions/28997441/hive-startup-error-terminal-initialization-failed-falling-back-to-unsupporte) related to different `jline` library versions in Hive and Hadoop. It can be resolved by setting `HADOOP_USER_CLASSPATH_FIRST=true` environment variable."
}
[/block]
For convenience you can create a simple script that will properly set all required variables and run Hive, like this:
[block:code]
{
  "codes": [
    {
      "code": "# Specify Hive home directory:\nexport HIVE_HOME=<Hive installation directory>\n\n# Specofy configuration files location:\nexport HIVE_CONF_DIR=<Path to our configuration folder>\n\n# If you did not set hadoop executable in PATH, specify Hadoop home explicitly:\nexport HADOOP_HOME=<Hadoop installation folder>\n\n# Avoid problem with different 'jline' library in Hadoop: \nexport HADOOP_USER_CLASSPATH_FIRST=true\n\n${HIVE_HOME}/bin/hive \"${@}\"",
      "language": "shell"
    }
  ]
}
[/block]
This script can be used to start Hive interactive console:
[block:code]
{
  "codes": [
    {
      "code": "$ hive-ig cli\nhive> show tables;\nOK\nu_data\nTime taken: 0.626 seconds, Fetched: 1 row(s)\nhive> quit;\n$",
      "language": "text"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Starting HiveServer2"
}
[/block]
You may also want to use [HiveServer2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2) for enhanced client features. To start it you can also use the script created above:
[block:code]
{
  "codes": [
    {
      "code": "hive-ig --service hiveserver2",
      "language": "shell"
    }
  ]
}
[/block]
After the server is started, you can connect to it with any available [client](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients) (e.g., `beeline`). As a remote client, `beeline` can be run from any host, and it does not require any specific environment to work with "Ignited" Hive. Here is the example:
[block:code]
{
  "codes": [
    {
      "code": "$ ./beeline \nBeeline version 1.2.1 by Apache Hive\nbeeline> !connect jdbc:hive2://localhost:10000 scott tiger org.apache.hive.jdbc.HiveDriver\nConnecting to jdbc:hive2://localhost:10000\nConnected to: Apache Hive (version 1.2.1)\nDriver: Hive JDBC (version 1.2.1)\nTransaction isolation: TRANSACTION_REPEATABLE_READ\n0: jdbc:hive2://localhost:10000> show tables;\n+-----------+--+\n| tab_name  |\n+-----------+--+\n| u_data    |\n+-----------+--+\n1 row selected (0.957 seconds)\n0: jdbc:hive2://localhost:10000>",
      "language": "text"
    }
  ]
}
[/block]