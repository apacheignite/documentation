* [Query Cancellation](#query-cancellation)
* [Custom SQL Functions](#custom-sql-functions)
[block:api-header]
{
  "type": "basic",
  "title": "Query Cancellation"
}
[/block]
There are two ways to stop long running SQL queries in Ignite that can be caused, for instance, due to non-optimal indexes configuration.

The first approach is to set a query execution timeout for a specific `SqlQuery` or `SqlFieldsQuery`.
[block:code]
{
  "codes": [
    {
      "code": "SqlQuery qry = new SqlQuery<AffinityKey<Long>, Person>(Person.class, joinSql);\n\n// Setting query execution timeout\nqry.setTimeout(10_000, TimeUnit.SECONDS);",
      "language": "java"
    }
  ]
}
[/block]
The second approach in regards to the query cancellation is to halt the query by using `QueryCursor.close()`.
[block:code]
{
  "codes": [
    {
      "code": "SqlQuery qry = new SqlQuery<AffinityKey<Long>, Person>(Person.class, joinSql);\n\n// Getting query cursor.\nQueryCursor<List> cursor = cache.query(qry);\n        \n// Executing query.\n....\n        \n// Halting the query that might be still in progress.\ncursor.close();",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Query cancellation API is supported in Ignite 1.8 and later versions."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Custom SQL Functions"
}
[/block]
Apache Ignite SQL Engine allows extending SQL functions' set, defined by ANSI-99 specification, by an addition of custom SQL functions written in Java.

A custom SQL function is no more than a public static method marked by `@QuerySqlFunction` annotation.
[block:code]
{
  "codes": [
    {
      "code": "// Defining a custom SQL function.\npublic class MyFunctions {\n    @QuerySqlFunction\n    public static int sqr(int x) {\n        return x * x;\n    }\n}",
      "language": "java"
    }
  ]
}
[/block]
A class that owns the custom SQL function has to be registered in a specific `CacheConfiguration` using  `setSqlFunctionClasses(...)` method.
[block:code]
{
  "codes": [
    {
      "code": "// Preparing a cache configuration.\nCacheConfiguration cfg = new CacheConfiguration();\n\n// Registering the class that contains custom SQL functions.\ncfg.setSqlFunctionClasses(MyFunctions.class);\n            ",
      "language": "java"
    }
  ]
}
[/block]
 After a cache with the configuration above is deployed, you're free to call the custom function from SQL queries like it's shown below.
[block:code]
{
  "codes": [
    {
      "code": "// Preparing the query that uses customly defined 'sqr' function.\nSqlFieldsQuery query = new SqlFieldsQuery(\n  \"SELECT name FROM Blocks WHERE sqr(size) > 100\");\n\n// Executing the query.\ncache.query(query).getAll();        ",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Classes that are registered with `CacheConfiguration.setSqlFunctionClasses(...)` have to be added to the classpath of all the nodes where defined custom functions might be executed. Otherwise, you'll get a `ClassNotFoundException` during custom function's execution time."
}
[/block]