* [JDBC Connection](#jdbc-connection)
* [Data Streaming](#data-streaming)
* [Example](#example)
* [Backward Compatibility](#backward-compatibility)
[block:api-header]
{
  "type": "basic",
  "title": "JDBC Connection"
}
[/block]
Ignite is shipped with a JDBC driver that allows you to retrieve distributed data from caches using standard SQL queries and update the data using DML statements like `INSERT`, `UPDATE` or `DELETE` directly from the JDBC API side. 

In Ignite, the JDBC connection URL has the following pattern:
[block:code]
{
  "codes": [
    {
      "code": "jdbc:ignite:cfg://[<params>@]<config_url>",
      "language": "xml"
    }
  ]
}
[/block]
* `<config_url>` is required and represents any valid URL that points to an Ignite configuration file for Ignite client node. This node will be started within the Ignite JDBC Driver when it (JDBC driver) tries to establish a connection with the cluster. The JDBC driver will forward the SQL queries, sent by the user application, to the cluster via the client node.
* `<params>` is optional and has the following format:
[block:code]
{
  "codes": [
    {
      "code": "param1=value1:param2=value2:...:paramN=valueN",
      "language": "text"
    }
  ]
}
[/block]
The following parameters are supported:
[block:parameters]
{
  "data": {
    "h-0": "Properties",
    "h-1": "Description",
    "h-2": "Default",
    "0-0": "`cache`",
    "1-0": "`nodeId`",
    "2-0": "`local`",
    "3-0": "`collocated`",
    "0-1": "Cache name. If it is not defined the default cache will be used. Note that the cache name is case sensitive.",
    "1-1": "ID of node where query will be executed. It can be useful for querying through local caches.",
    "2-1": "Query will be executed only on a local node. Use this parameter with `nodeId` parameter in order to limit data set by specified node.",
    "2-2": "false",
    "3-1": "Flag that is used for optimization purposes. Whenever Ignite executes a distributed query, it sends sub-queries to individual cluster members. If you know in advance that the elements of your query selection are collocated together on the same node, Ignite can make significant performance and network optimizations.",
    "3-2": "false",
    "4-0": "`distributedJoins`",
    "4-1": "Allows use distributed joins for non collocated data.",
    "4-2": "false",
    "5-0": "`streaming`",
    "5-1": "Turns on bulk data load mode via `INSERT` statements for this connection.",
    "5-2": "false",
    "6-0": "`streamingAllowOverwrite`",
    "6-1": "Tells Ignite to overwrite values for existing keys on duplication instead of skipping them. Refer to [IgniteDataStreamer JavaDoc](https://ignite.apache.org/releases/1.8.0/javadoc/org/apache/ignite/IgniteDataStreamer.html) for more details.",
    "6-2": "false",
    "7-0": "`streamingFlushFrequency`",
    "7-1": "Timeout, in _milliseconds_, that data streamer should use to flush data. By default, *the data is flushed on connection close*. Refer to [IgniteDataStreamer JavaDoc](https://ignite.apache.org/releases/1.8.0/javadoc/) for more details.",
    "7-2": "0",
    "8-0": "`streamingPerNodeBufferSize`",
    "8-1": "Data streamer's per node buffer size. Refer to [IgniteDataStreamer JavaDoc](https://ignite.apache.org/releases/1.8.0/javadoc/org/apache/ignite/IgniteDataStreamer.html) for more details.",
    "8-2": "1024",
    "9-0": "`streamingPerNodeParallelOperations`",
    "9-1": "Data streamer's per node parallel operations number. Refer to [IgniteDataStreamer JavaDoc](https://ignite.apache.org/releases/1.8.0/javadoc/org/apache/ignite/IgniteDataStreamer.html) for more details.",
    "9-2": "16"
  },
  "cols": 3,
  "rows": 10
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "By default, all Ignite nodes are started as server nodes, and client mode needs to be explicitly enabled. However, regardless of the configuration, the JDBC driver always starts a node in client mode. See [Clients and Servers](doc:clients-vs-servers) section for details.",
  "title": "Client vs Server Nodes"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Cross-Cache Queries",
  "body": "Cache that the driver is connected to is treated as the default schema. To query across multiple caches, [Cross-Cache Query](/docs/cache-queries#cross-cache-queries) functionality can be used."
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Joins and Collocation",
  "body": "Just like with [Cache SQL Queries](doc:cache-queries) used from `IgniteCache` API, joins on `PARTITIONED` caches will work correctly only if joined objects are stored in collocated mode. Refer to [Affinity Collocation](/docs/affinity-collocation#collocate-data-with-data) for more details."
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Replicated vs Partitioned Caches",
  "body": "Queries on `REPLICATED` caches will run directly only on one node, while queries on `PARTITIONED` caches are distributed across all cache nodes."
}
[/block]

[block:api-header]
{
"type": "basic",
  "title": "Data Streaming"
}
[/block]
Ignite is able to load data in bulk via SQL by associating a *data streamer* with JDBC connection and feeding all incoming data to it. To achieve that, it's enough to set `streaming` param to `true` for a connection like this:
[block:code]
{
  "codes": [
    {
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is not specified, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite:cfg://streaming=true@file:///etc/config/ignite-jdbc.xml\");",
      "language": "java"
    }
  ]
}
[/block]
The connection created in such way **does not permit any SQL operations besides `INSERT`**, so the suggested pattern to use data streaming in your app is just to create a separate connection for bulk data load. Currently, Ignite JDBC driver has 5 connection params that affect the work of data streamer, please see their names and description in the table above together with those for other params. Those params cover basically all settings of a classic `IgniteDataStreamer` and allow you to do fine tuning of the streamer according to your needs. Please refer to [Data Streamers](doc:data-streamers) section of Ignite docs for more info on how to configure a streamer.
[block:callout]
{
  "type": "info",
  "title": "Stream flush",
  "body": "By default, streamed data is flushed only **on connection close**, so if you need that to happen more often, please set flush timeout accordingly via `streamingFlushFrequency` connection param. Still, if your app needs to know precisely the moment when all data definitely is in cache, just wait until standard JDBC `Connection`'s close completes for an Ignite connection as shown in the following example."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is not specified, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite:cfg://streaming=true@file:///etc/config/ignite-jdbc.xml\");\n\nPreparedStatement stmt = conn.prepareStatement(\"INSERT INTO Person(_key, name, \t\t\t\tage) VALUES(CAST(? as BIGINT), ?, ?)\");\n\n// Let's load ourselves a bunch of John Smiths\nfor (int i = 1; i < 100000; i++) {\n      // Insert a Person with a Long key.\n      stmt.setInt(1, i);\n      stmt.setString(2, \"John Smith\");\n      stmt.setInt(3, 25);\n  \n  \t\tstmt.execute();\n}\n\nconn.close();\n\n// Beyond this point, all data is guaranteed to be in the cache.",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
Ignite JDBC driver automatically gets only those fields that you actually need from objects stored in the cache. Let's say you have a `Person` class declared like this:
[block:code]
{
  "codes": [
    {
      "code": "public class Person {\n    @QuerySqlField\n    private String name;\n \n    @QuerySqlField\n    private int age;\n \n    // Getters and setters.\n    ...\n}",
      "language": "java"
    }
  ]
}
[/block]
If you have instances of this class in a cache, you can query individual fields (name, age or both) via the standard JDBC API, like so:
[block:code]
{
  "codes": [
    {
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is not specified, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite:cfg://file:///etc/config/ignite-jdbc.xml\");\n \n// Query names of all people.\nResultSet rs = conn.createStatement().executeQuery(\"select name from Person\");\n \nwhile (rs.next()) {\n    String name = rs.getString(1);\n    ...\n}\n \n// Query people with specific age using prepared statement.\nPreparedStatement stmt = conn.prepareStatement(\"select name, age from Person where age = ?\");\n \nstmt.setInt(1, 30);\n \nResultSet rs = stmt.executeQuery();\n \nwhile (rs.next()) {\n    String name = rs.getString(\"name\");\n    int age = rs.getInt(\"age\");\n    ...\n}",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
Moreover, you can modify the data with the usage of DML statements.

##INSERT 
[block:code]
{
  "codes": [
    {
      "code": "// Insert a Person with a Long key.\nPreparedStatement stmt = conn.prepareStatement(\"INSERT INTO Person(_key, name, age) VALUES(CAST(? as BIGINT), ?, ?)\");\n \nstmt.setInt(1, 1);\nstmt.setString(2, \"John Smith\");\nstmt.setInt(3, 25);\n\nstmt.execute();",
      "language": "java"
    }
  ]
}
[/block]
##MERGE
[block:code]
{
  "codes": [
    {
      "code": "// Merge a Person with a Long key.\nPreparedStatement stmt = conn.prepareStatement(\"MERGE INTO Person(_key, name, age) VALUES(CAST(? as BIGINT), ?, ?)\");\n \nstmt.setInt(1, 1);\nstmt.setString(2, \"John Smith\");\nstmt.setInt(3, 25);\n \nstmt.executeUpdate();",
      "language": "java"
    }
  ]
}
[/block]
##UPDATE
[block:code]
{
  "codes": [
    {
      "code": "// Update a Person.\nconn.createStatement().\n  executeUpdate(\"UPDATE Person SET age = age + 1 WHERE age = 25\");",
      "language": "java"
    }
  ]
}
[/block]
##DELETE
[block:code]
{
  "codes": [
    {
      "code": "conn.createStatement().execute(\"DELETE FROM Person WHERE age = 25\");",
      "language": "java"
    }
  ]
}
[/block]
A minimalistic version of `ignite-jdbc.xml` configuration file might look like the one below:
[block:code]
{
  "codes": [
    {
      "code": "<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\">\n    <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n        <property name=\"clientMode\" value=\"true\"/>\n\n        <property name=\"peerClassLoadingEnabled\" value=\"true\"/>\n\n        <!-- Configure TCP discovery SPI to provide list of initial nodes. -->\n        <property name=\"discoverySpi\">\n            <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n                <property name=\"ipFinder\">\n                    <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.multicast.TcpDiscoveryMulticastIpFinder\"/>\n                </property>\n            </bean>\n        </property>\n    </bean>\n</beans>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Backward compatibility"
}
[/block]
For previous versions of Ignite (prior 1.4) JDBC connection URL has the following pattern:
[block:code]
{
  "codes": [
    {
      "code": "jdbc:ignite://<hostname>:<port>/<cache_name>",
      "language": "xml"
    }
  ]
}
[/block]
See the corresponding [documentation](https://apacheignite.readme.io/v1.3/docs/jdbc-driver) for details.