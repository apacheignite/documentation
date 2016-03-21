Ignite now includes ODBC driver that allows you to retrieve distributed data from cache using standard SQL queries and native ODBC API.

For detailed info on ODBC please refer to [ODBC Programmer's Reference](https://msdn.microsoft.com/en-us/library/ms714177.aspx).
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite ODBC driver implements version 3.0 of the ODBC API.
[block:api-header]
{
  "type": "basic",
  "title": "Prerequisites"
}
[/block]
Apache Ignite ODBC Driver was officially tested on:
[block:parameters]
{
  "data": {
    "0-0": "OS",
    "0-1": "Windows (XP and up, both 32-bit and 64-bit versions), \nWindows Server (2008 and up, both 32-bit and 64-bit versions)\nUbuntu (14.x and 15.x 64-bit)",
    "1-0": "C++ compiler",
    "1-1": "MS Visual C++ (10.0 and up), g++ (4.4.0 and up)",
    "2-1": "2010 and above",
    "2-0": "Visual Studio"
  },
  "cols": 2,
  "rows": 3
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Building ODBC driver"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Quering data"
}
[/block]
ODBC Driver internally uses Fields queries to retrieve data from the Apache Ignite cache. This means that by ODBC you can only access those fields that are [accessible for SQL queries](/docs/sql-queries#section-making-fields-visible-for-sql-queries).

Below you can see an example of two classes that can be queried by the ODBC Driver:
[block:code]
{
  "codes": [
    {
      "code": "/** All fields of the class will be visible in SQL. */\npublic class Person {\n\t@QuerySqlField\n  private long id;\n  \n  @QuerySqlField\n  public Long orgId;\n  \n  @QuerySqlField\n  private String name;\n  \n  @QuerySqlField\n  private int age;\n  \n  public Organization(String name) {\n    id = ID_GEN.incrementAndGet();\n\n    this.name = name;\n  }\n}",
      "language": "java",
      "name": "Person"
    },
    {
      "code": "/** All fields of the class will be visible in SQL. */\npublic class Organization {\n  @QuerySqlField\n  private Long id;\n\n  @QuerySqlField\n  private String name;\n}",
      "language": "java",
      "name": "Organization"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Predefined Fields",
  "body": "In addition to all the fields marked with `@QuerySqlField` annotation, each table will have two special predefined fields: `_key` and `_val`, which represent links to whole key and value objects. This is useful, for example, when one of them is a primitive and you want to filter by its value. To do this, execute a query like `SELECT * FROM Person WHERE _key = 100`."
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