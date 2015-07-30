Install [Apache Hive](http://hive.apache.org/).

We assume that Hadoop is already installed and configured to run over Ignite cluster.

To configure Hive to run over such Hadoop, we need the following.
- Hive runs hadoop executable. So, to run over "ignited" Hadoop Hive should know where the correct `hadoop` executable is located. This can be done either with adding appropriate `hadoop` executable into `PATH` environment variable (this executable should anyway be located in a folder named `bin/`), or with specifying `HADOOP_HOME` environemnt variable, which overrides `hadoop` in the path.
 
- We need to direct Hive to use specific Hadoop configuration. To do that you can have all the configuration files (typically they are `core-site.xml`, `hive-site.xml`, `mapred-site.xml`) in one directory, and specify it with `HIVE_CONF_DIR` environment variable. 

- We recommend to use Hive template configuration file `<IGNITE_HOME>/config/hadoop/hive-site.ignite.xml` to get Ignite specific settings. 
 
- We may also need to overcome possible [problem](http://stackoverflow.com/questions/28997441/hive-startup-error-terminal-initialization-failed-falling-back-to-unsupporte) related to different **jline** library versions in Hive and Hadoop: in our case this can be solved by setting `HADOOP_USER_CLASSPATH_FIRST=true`.

Summing the above, we can write small Hive launcher script (e.g. `hive-ig`):
[block:code]
{
  "codes": [
    {
      "code": "# Specify Hive home directory:\nexport HIVE_HOME=<Hive installation directory>\n\n# Define specific configuration location:\nexport HIVE_CONF_DIR=<Path to our configuration folder>\n\n# If we do not use hadoop executable in PATH, specify Hadoop home explicitly:\nexport HADOOP_HOME=<Hadoop installation folder>\n\n# Avoid problem with different 'jline' library in Hadoop, see \n# http://stackoverflow.com/questions/28997441/hive-startup-error-terminal-initialization-failed-falling-back-to-unsupporte : \nexport HADOOP_USER_CLASSPATH_FIRST=true\n\n${HIVE_HOME}/bin/hive \"${@}\"",
      "language": "shell"
    }
  ]
}
[/block]
After that we should be able to start Hive interactive console:
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
 We can also start [Hive Server 2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2) to have enhanced client features. To do that start our Hive launcher script as follows:
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
After that we can connect to Hive Server 2 with a [client](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients) e.g. with `beeline`:
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