--------------
title: JDBC Driver
excerpt: Connect to Ignite using standard JDBC driver.
--------------

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
      "code": "jdbc:ignite://<hostname>:<port>/<cache_name>",
      "language": "xml",
      "name": "XML"
    }
  ]
}
[/block]
  * Hostname is required.
  * If port is not defined, `11211` is used (default for Ignite client).
  * Leave <cache_name> empty if you are connecting to a default cache. Note that the cache name is case sensitive.
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
  "title": "Configuration"
}
[/block]
Internally, JDBC driver is based on Ignite Java client. Therefore, all client configuration properties can be applied to JDBC connection. Client configuration properties can be defined in `Properties` object passed to `DriverManager.getConnection(String, Properties)` method. Possible properties are: 
[block:parameters]
{
  "data": {
    "0-0": "`ignite.client.protocol`",
    "h-0": "Properties",
    "h-1": "Description",
    "h-2": "Default",
    "0-1": "Communication protocol (TCP or HTTP).",
    "0-2": "TCP",
    "1-0": "`ignite.client.connectTimeout`",
    "1-1": "Socket connection timeout.",
    "1-2": "0 (infinite timeout)",
    "2-0": "`ignite.client.tcp.noDelay`",
    "3-0": "`ignite.client.ssl.enabled`",
    "4-0": "`ignite.client.ssl.protocol`",
    "5-0": "`ignite.client.ssl.key.algorithm`",
    "6-0": "`ignite.client.ssl.keystore.location`",
    "7-0": "`ignite.client.ssl.keystore.password`",
    "9-0": "`ignite.client.ssl.truststore.location`",
    "10-0": "`ignite.client.ssl.truststore.password`",
    "11-0": "`ignite.client.ssl.truststore.type`",
    "12-0": "`ignite.client.credentials`",
    "13-0": "`ignite.client.cache.top`",
    "14-0": "`ignite.client.topology.refresh`",
    "2-1": "Flag indicating whether TCP_NODELAY flag should be enabled for outgoing connections.",
    "2-2": "true",
    "3-1": "Flag indicating that SSL is needed for connection. If SSL is enabled, `ignite.client.ssl.keystore.location` and `ignite.client.ssl.truststore.location` properties are required.",
    "3-2": "false",
    "4-1": "SSL protocol (SSL or TLS).",
    "4-2": "TLS",
    "5-1": "Key manager algorithm.",
    "5-2": "SunX509",
    "6-1": "Key store to be used by client to connect with GridGain topology. This property `is required` if SSL is enabled.",
    "7-1": "Key store password.",
    "15-0": "`ignite.client.idleTimeout`",
    "8-0": "`ignite.client.ssl.keystore.type`",
    "8-1": "Key store type.",
    "8-2": "jks",
    "9-1": "Trust store to be used by client to connect with GridGain topology. This property `is required` if SSL is enabled.",
    "10-1": "Trust store password.",
    "11-1": "Trust store type.",
    "12-1": "Client credentials used in authentication process.",
    "13-1": "Flag indicating that topology is cached internally. Cache will be refreshed in the background with interval defined by `ignite.client.topology.refresh` property (see below).",
    "13-2": "false",
    "11-2": "jks",
    "14-1": "Topology cache refresh frequency (ms).",
    "14-2": "2000 ms",
    "15-1": "Maximum amount of time that connection can be idle before it is closed (ms).",
    "15-2": "30000 ms"
  },
  "cols": 3,
  "rows": 16
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
      "code": "// Register JDBC driver.\nClass.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n \n// Open JDBC connection (cache name is empty, which means that we use default cache).\nConnection conn = DriverManager.getConnection(\"jdbc:ignite://localhost:11211/\");\n \n// Query names of all people.\nResultSet rs = conn.createStatement().executeQuery(\"select name from Person\");\n \nwhile (rs.next()) {\n    String name = rs.getString(1);\n    ...\n}\n \n// Query people with specific age using prepared statement.\nPreparedStatement stmt = conn.prepareStatement(\"select name, age from Person where age = ?\");\n \nstmt.setInt(1, 30);\n \nResultSet rs = stmt.executeQuery();\n \nwhile (rs.next()) {\n    String name = rs.getString(\"name\");\n    int age = rs.getInt(\"age\");\n    ...\n}",
      "language": "java"
    }
  ]
}
[/block]