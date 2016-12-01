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
  "title": "Configuration Properties"
}
[/block]
There is a number of SQL queries related properties that you can adjust in order to influence on queries execution behavior.

The properties are divided into global ones that are exposed over `CacheConfiguration` and affect all the queries that will be executed over this cache and per-query properties.

## Cache Configuration Properties
 
[block:parameters]
{
  "data": {
    "h-0": "Property Name",
    "h-1": "Description",
    "h-2": "Default Value",
    "0-0": "`setSqlSchema(...)`",
    "0-1": "Sets sql schema to be used for current cache. This name will correspond to SQL ANSI-99 standard. Nonquoted identifiers are not case sensitive. Quoted identifiers are case sensitive.",
    "0-2": "Cache name.",
    "1-0": "`setSqlEscapeAll(...)`",
    "1-1": "If set to `true`, all the SQL table and field names will be escaped with double quotes like `\"tableName\".\"fieldsName\"`. This enforces case sensitivity for field names and also allows having special characters in table and field names.",
    "1-2": "`false`",
    "2-0": "`setSqlOnheapRowCacheSize(...)`",
    "2-1": "Defines a number of SQL rows which will be cached onheap to avoid deserialization on each SQL index access. This setting only makes sense when offheap is enabled for this cache. Refer to [indexes](doc:indexes) page for more details on type of indexes.",
    "2-2": "`10,240`"
  },
  "cols": 3,
  "rows": 3
}
[/block]
## `SqlFields` and `SqlFieldsQuery` Configuration Properties
[block:parameters]
{
  "data": {
    "h-0": "Property Name",
    "h-1": "Description",
    "h-2": "Default Value",
    "h-3": "Supported Query Type",
    "0-0": "`setCollocated(...)`",
    "0-1": "Collocation flag is used for optimization purposes. Whenever Ignite executes a distributed query, it sends sub-queries to individual cluster members. If you know in advance that the elements of your query selection are collocated together on the same node Ignite can make significant performance and network optimizations by grouping data on remote nodes.",
    "0-2": "`false`",
    "1-0": "`setDistributedJoins(...)`",
    "1-1": "Enables distributed non-collocated mode for a given query.",
    "1-2": "`false`",
    "2-0": "`setEnforceJoinOrder(...)`",
    "2-1": "Sets flag to enforce join order of tables in the query. If set to `true`  the query optimizer will not reorder tables in a join clause.",
    "2-2": "`false`",
    "3-0": "`setLocal(...)`",
    "3-1": "Forces query execution in purely local mode. Refer to [local queries](doc:local-queries) page for more details on this mode.",
    "3-2": "`false`",
    "4-0": "`setPageSize(...)`",
    "4-1": "Defines a maximum number of entries that can be transferred in a single response chunk to a reducing node (query initiator).",
    "4-2": "`1024`",
    "5-0": "`setTimeout(...)`",
    "5-1": "Sets the query execution timeout. Query will be automatically cancelled if the execution timeout is exceeded. Disabled by default. \n\nAvailable in Apache Ignite 1.8 and later versions.",
    "5-2": "`0`"
  },
  "cols": 3,
  "rows": 6
}
[/block]