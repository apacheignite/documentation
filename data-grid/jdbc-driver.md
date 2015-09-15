Ignite is shipped with JDBC driver that allows you to retrieve distributed data from cache using standard SQL queries and JDBC API. 
[block:api-header]
{
  "type": "basic",
  "title": "JDBC Connection"
}
[/block]
In Ignite, JDBC connection URL has the following pattern:
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
* `<config_url>` is required and represents any valid URL which points to Ignite configuration file. See [Clients and Servers](doc:clients-vs-servers) section for details.
* `<params>` is optional part and have the following format:
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
    "0-1": "Cache name. If it is not defined than default cache will be used. Note that the cache name is case sensitive.",
    "1-1": "ID of node where query will be executed. It can be useful for querying through local caches.",
    "2-1": "Query will be executed only on local node. Use this parameter with `nodeId` parameter in order to limit data set by specified node.",
    "2-2": "false",
    "3-1": "Flag that used for optimization purposes. Whenever Ignite executes a distributed query, it sends sub-queries to individual cluster members. If you know in advance that the elements of your query selection are collocated together on the same node, Ignite can make significant performance and network optimizations.",
    "3-2": "false"
  },
  "cols": 3,
  "rows": 4
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
Ignite JDBC driver automatically gets only those fields that you actually need from objects stored in cache. For example, let's say you have Person class declared like this:
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
If you have instances of this class in cache, you can query individual fields (name, age or both) via standard JDBC API, like so:
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

[block:api-header]
{
  "type": "basic",
  "title": "Backward compatibility"
}
[/block]
For previous versions of Ignite (less than 1.4) JDBC connection URL has the following pattern:
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