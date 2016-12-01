* [Query Cancellation](#query-cancellation)
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
The second approach in regards to the query cancellation is to halt the query with the usage of `QueryCursor.close()`.
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