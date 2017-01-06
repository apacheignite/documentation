In some scenarios, SQL Grid can fall back from query execution in the distributed mode to the local mode. In the local mode, a query is simply passed to the underlying H2 engine which runs it against a node's local data set only.
 
Those scenarios are the following:
* If a query is executed against a `REPLICATED` cache on a node where the cache is deployed then Ignite assumes that all the data is available locally and will implicitly run a simple local SQL query instead.
* A query is executed against a `LOCAL` cache.
* The local mode is explicitly enabled for a query with the usage of `SqlQuery.setLocal(true)` or `SqlFieldsQuery.setLocal(true)`.

The first two scenarios will provide you with a complete and consistent result set all the time even if a cluster topology changes (new node joins a cluster or an old one leaves it) at the time a query is being executed.

However, the third scenario when the local mode is enabled explicitly by your application code has to be used with a precaution. The reason is that if you decide to run a local SQL query against a `PARTITIONED` cache on some of the nodes and the topology changes during the query execution time then you might get a partial result set because data rebalancing will be triggered in parallel. The SQL engine doesn't handle this specific situation and if you still need to execute a local SQL query against a `PARTITIONED` cache then you should execute the query as a part of `affinityRun(...)` or `affinityCall(...)` methods as it's described [here](http://apacheignite.gridgain.org/docs/collocate-compute-and-data#affinity-call-and-run-methods).