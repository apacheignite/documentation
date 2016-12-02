In some scenarios, SQL Grid can fall back from query execution in the distributed mode to the local mode. In the local mode a query is simply passed to the underlying H2 engine which runs it against a node's local data set only.
 
Those scenarios are the following:
* If a query is executed against a `REPLICATED` cache on a node where the cache is deployed then Ignite assumes that all the data is available locally and will implicitly run a simple local SQL query instead.
* A query is executed against a `LOCAL` cache.
* The local mode is explicitly enabled for a query with the usage of `SqlQuery.setLocal(true)` or `SqlFieldsQuery.setLocal(true)`