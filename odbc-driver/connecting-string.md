Apache Ignite ODBC driver supports and uses following connection string/DSN
arguments:
[block:parameters]
{
  "data": {
    "h-0": "Parameter",
    "h-1": "Description",
    "h-2": "Default value",
    "0-0": "SERVER",
    "1-0": "PORT",
    "2-0": "CACHE",
    "2-1": "Cache name. If it is not defined than default cache will be used. Note that the cache name is case sensitive.",
    "1-1": "Port on which OdbcProcessor of the node is listening.",
    "0-1": "Address of the node to connect to.",
    "0-2": "lacalhost",
    "1-2": "11443"
  },
  "cols": 3,
  "rows": 3
}
[/block]
All parameter names are case-insensitive so `SERVER`, `Server` and `server` all are
valid parameter names and refer to the same parameter.
[block:callout]
{
  "type": "info",
  "body": "Cache that the driver is connected to is treated as the default schema. To query across multiple caches, [Cross-Cache Query](/docs/cache-queries#cross-cache-queries) functionality can be used.",
  "title": "Cross-Cache Queries"
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Just like with [Cache SQL Queries](doc:cache-queries) used from `IgniteCache` API, joins on `PARTITIONED` caches will work correctly only if joined objects are stored in collocated mode. Refer to [Affinity Collocation](/docs/affinity-collocation#collocate-data-with-data) for more details.",
  "title": "Joins and Collocation"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Replicated vs Partitioned Caches",
  "body": "Queries on `REPLICATED` caches will run directly only on one node, while queries on `PARTITIONED` caches are distributed across all cache nodes."
}
[/block]
## Connecting string samples
You can find samples of the connecting string below. These strings can be used with `SQLDriverConnect` ODBC call.
[block:code]
{
  "codes": [
    {
      "code": "DRIVER={Apache Ignite};SERVER=localhost;PORT=11443;CACHE=MyCache",
      "language": "text",
      "name": "Specific cache"
    },
    {
      "code": "DRIVER={Apache Ignite};SERVER=localhost;PORT=11443",
      "language": "text",
      "name": "Default cache"
    },
    {
      "code": "DRIVER={Apache Ignite}",
      "language": "text",
      "name": "All defaults"
    }
  ]
}
[/block]