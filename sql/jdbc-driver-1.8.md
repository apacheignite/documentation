* [JDBC Connection](#jdbc-connection)
* [Example](#example)
* [Backward Compatibility](#backward-compatibility)
[block:api-header]
{
  "type": "basic",
  "title": "JDBC Connection"
}
[/block]
Ignite is shipped with a JDBC driver that allows you to retrieve distributed data from caches using standard SQL queries and the JDBC API. 

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
* `<config_url>` is required and represents any valid URL which points to an Ignite configuration file for Ignite client node that will be started during connection establishing by JDBC driver. See [Clients and Servers](doc:clients-vs-servers) section for details.
* `<params>` is an optional part and has the following format:
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
    "4-2": "false"
  },
  "cols": 3,
  "rows": 5
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
  "title": "Example"
}
[/block]
Ignite JDBC driver automatically gets only those fields that you actually need from objects stored in cache. For example if you have a `Person` class declared like this:
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
If you have instances of this class in a cache, you can query individual fields (name, age or both) via standard JDBC API, like so:
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
Minimalistic version of `ignite-jdbc.xml` configuration file can look like:
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
  "title": "Modifying data with DML"
}
[/block]
Starting with 1.8.0, Ignite supports DML operations - namely, **INSERT**, **MERGE**, **UPDATE**, and **DELETE**. More information on them is in [DML doc](doc:dml), here we'll showcase their usage from JDBC driver using model class `Person` from above and different methods from JDBC `Statement` and `PreparedStatement` interfaces.

##INSERT
[block:code]
{
  "codes": [
    {
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is not specified, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite:cfg://file:///etc/config/ignite-jdbc.xml\");\n \n// Insert a Person with a Long key\nPreparedStatement stmt = conn.prepareStatement(\"INSERT INTO Person(_key, name, age) VALUES(CAST(? as BIGINT), ?, ?)\");\n \nstmt.setInt(1, 1);\nstmt.setString(2, \"John Smith\");\nstmt.setInt(3, 25);\n \nboolean res = stmt.execute();\n\n// Has to be false as we've executed non query operation\nSystem.out.println(res);",
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
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is not specified, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite:cfg://file:///etc/config/ignite-jdbc.xml\");\n \n// Merge a Person with a Long key\nPreparedStatement stmt = conn.prepareStatement(\"MERGE INTO Person(_key, name, age) VALUES(CAST(? as BIGINT), ?, ?)\");\n \nstmt.setInt(1, 1);\nstmt.setString(2, \"John Smith\");\nstmt.setInt(3, 25);\n \nint res = stmt.executeUpdate();\n\n// Has to be 1\nSystem.out.println(res);",
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
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is not specified, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite:cfg://file:///etc/config/ignite-jdbc.xml\");\n \n// Merge a Person with a Long key\nint res = conn.createStatement().executeUpdate(\"UPDATE Person SET age = age + 1 WHERE age = 25\");\n\n// Will print number of affected items\nSystem.out.println(res);",
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
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is not specified, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite:cfg://file:///etc/config/ignite-jdbc.xml\");\n \n// Merge a Person with a Long key\nboolean res = conn.createStatement().execute(\"DELETE FROM Person WHERE age = 25\");\n\n// Has to be false as we've executed non query operation\nSystem.out.println(res);",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "title": "Batch updates are not supported",
  "body": "This most likely will change in the near future via performing query rewriting prior to sending query task to Ignite cluster."
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