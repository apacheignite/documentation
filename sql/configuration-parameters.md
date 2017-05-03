There are a number of SQL queries related properties that you can adjust in order to influence the query execution behavior.

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
    "2-2": "`10,240`",
    "3-0": "`setSnapshotableIndex(...)`",
    "3-1": "Enables snapshotable indexes implementation for indexed data stored in Java heap. Refer to [this documentation section](https://apacheignite.readme.io/docs/indexes#skiplist-based-and-snapshotable-indexes) for more details.",
    "3-2": "`false`"
  },
  "cols": 3,
  "rows": 4
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
    "0-1": "Collocation flag is used for optimization purposes of queries with GROUP BY statements. Whenever Ignite executes a distributed query, it sends sub-queries to individual cluster members. If you know in advance that the elements of your query selection are collocated together on the same node and you group by collocated key (primary or affinity key), then Ignite can make significant performance and network optimizations by grouping data on remote nodes.",
    "0-2": "`false`",
    "1-0": "`setDistributedJoins(...)`",
    "1-1": "Enables distributed non-collocated mode for a given query.",
    "1-2": "`false`",
    "2-0": "`setEnforceJoinOrder(...)`",
    "2-1": "Sets flag to enforce join order of tables in the query. If set to `true`  the query optimizer will not reorder tables in a join clause.",
    "2-2": "`false`",
    "4-0": "`setLocal(...)`",
    "4-1": "Forces query execution in purely local mode. Refer to [local queries](doc:local-queries) page for more details on this mode.",
    "4-2": "`false`",
    "5-0": "`setPageSize(...)`",
    "5-1": "Defines a maximum number of entries that can be transferred in a single response chunk to a reducing node (query initiator).",
    "5-2": "`1024`",
    "7-0": "`setTimeout(...)`",
    "7-1": "Sets the query execution timeout. Query will be automatically cancelled if the execution timeout is exceeded. Disabled by default. \n\nAvailable in Apache Ignite 1.8 and later versions.",
    "7-2": "`0`",
    "3-0": "`setReplicatedOnly(...)`",
    "3-1": "If a SQL query is executed over the data stored across replicated caches only, then you may want to set this parameter to `true`. This is a special hint to the SQL engine that might produce a more effective execution plan for the query.",
    "3-2": "`false`",
    "6-0": "`setPartitions(...)`",
    "6-1": "Sets partitions for a query execution. The query will be executed only on nodes which are primary for specified partitions.",
    "6-2": "`null` (ignored)",
    "8-0": "`setAlias(...)`",
    "8-1": "Sets alias that can be used as table name in a query string.",
    "8-2": "`null` (ignored)"
  },
  "cols": 3,
  "rows": 9
}
[/block]