In some scenarios, SQL Grid can fall back from query execution in the distributed mode to the local mode. In the local mode, a query is simply passed to the underlying H2 engine which runs it against a node's local data set only.
 
Those scenarios are as following:
* If a query is executed against a `REPLICATED` cache on a node where the cache is deployed, then Ignite assumes that all the data is available locally and will implicitly run a simple local SQL query instead.
* A query is executed against a `LOCAL` cache.
* The local mode is explicitly enabled for a query with the usage of `SqlQuery.setLocal(true)` or `SqlFieldsQuery.setLocal(true)`.

The first two scenarios will provide you with a complete and consistent result set regardless of cluster topology changes (new node joins the cluster, or an old one leaves it) at the time of query execution.

However, the third scenario where the local mode is enabled explicitly by your application code should be used with precaution. The reason is that if you decide to run a local SQL query against a `PARTITIONED` cache on some of the nodes and the topology changes during the query execution, then you might get a partial result set because data rebalancing will be triggered in parallel. The SQL engine does not handle this particular situation. If you still need to execute a local SQL query against a `PARTITIONED` cache, then you should execute the query as part of `affinityRun(...)` or `affinityCall(...)` methods as described [here](http://apacheignite.gridgain.org/docs/collocate-compute-and-data#affinity-call-and-run-methods).