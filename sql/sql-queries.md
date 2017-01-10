* [Overview](#overview)
* [How SQL Queries Work](#how-sql-queries-work)
* [Query Types](#query-types)
* [Cross-Cache Queries](#cross-cache-queries)
* [Distributed Joins](#distributed-joins)
* [Example](#example)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite supports free-form SQL queries without any limitations. The SQL syntax is ANSI-99 compliant which means that you can use any kind of SQL functions, aggregations, groupings or joins, defined by the specification, as a part of an SQL query.

Furthermore, the queries are fully distributed. The SQL engine is capable of not only mapping a query to specific nodes and reducing their responses into a final result set, it is also able to join data sets stored in different caches on various nodes. Additionally, the SQL engine performs in a fault-tolerant fashion guaranteeing that you will never get an incomplete or wrong result in case a new node joins the cluster or an old one leaves it.
[block:api-header]
{
  "type": "basic",
  "title": "How SQL Queries Work"
}
[/block]
Apache Ignite SQL Grid component is tightly coupled with [H2 Database](http://www.h2database.com) which, in short, is a fast in-memory and disk-based database written in Java and available under a number of open source licenses.

An embedded H2 instance is always started as a part of an Apache Ignite node process whenever `ignite-indexing` module is added to the node's classpath. Ignite leverages from H2's SQL query parser and optimizer as well as the execution planner. Lastly, H2 executes a query locally on a particular node (a distributed query is mapped to the node or the query is executed in `LOCAL` mode) and passes a local result to a distributed Ignite SQL engine for further processing. 

However, the data, as well as indexes, are always stored on the Ignite Data Grid side. Additionally, Ignite executes queries in a distributed and fault-tolerant manner which is not supported by H2.

Theoretically, Ignite SQL Grid executes queries in two ways:

First, if a query is executed against a `REPLICATED` cache on a node where the cache is deployed, then Ignite assumes that all the data is available locally and will run a simple local SQL query passing it directly to the H2 database engine. The same execution flow is true for `LOCAL` caches.
[block:callout]
{
  "type": "info",
  "title": "Local Queries",
  "body": "Learn more about local SQL queries in Ignite from [this page](doc:local-queries)."
}
[/block]
Second, if a query is executed over a `PARTITIONED` cache, then the execution flow will be the following:
* The query will be parsed and split into multiple map queries and a single reduce query.
* All the map queries are executed on all the data nodes where cache data resides.
* All the nodes provide result sets of local execution to the query initiator (reducing node) that, in turn, will accomplish the reduce phase by properly merging provided result sets.
[block:callout]
{
  "type": "info",
  "title": "Execution Flow of Cross-Cache Queries",
  "body": "The execution flow of cross-cache or join queries is not different from the one described for the `PARTITIONED` cache above and will be covered later as part of this documentation."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Query Types"
}
[/block]
There are two general types of SQL queries that are available at Java API level - `SqlQuery` and `SqlFieldsQuery`. 
[block:callout]
{
  "type": "info",
  "title": "Alternative APIs",
  "body": "Apache Ignite In-Memory SQL Grid is not bound to Java APIs only. You can connect to an Ignite cluster from .NET, C++, ODBC or JDBC sides and execute SQL queries without a need to be aware of Java APIs at all. Learn more about additional APIs from the pages listed below:\n* [.NET SQL Queries](https://apacheignite-net.readme.io/docs/sql-queries)\n* [C++ SQL Queries](https://apacheignite-cpp.readme.io/docs/sql-queries)\n* [JDBC Driver Based Queries](doc:jdbc-driver) \n* [ODBC Driver Based Queries](doc:quering-data)"
}
[/block]
## SqlQuery

`SqlQuery` is useful for scenarios when at the end of a query execution you need to get the whole object, stored in a cache (key and value), back in a final result set. The code snippet below shows how this can be done.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\nSqlQuery sql = new SqlQuery(Person.class, \"salary > ?\");\n\n// Find all persons earning more than 1,000.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(1000))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "SqlQuery"
    }
  ]
}
[/block]
## SqlFieldsQueries

Instead of selecting the whole object, you can choose to select only specific fields in order to minimize network and serialization overhead. For this purpose, Ignite implements a concept of `fields queries`. Basically, `SqlFieldsQuery` accepts a conventional ANSI-99 SQL query as its constructor​ parameter and executes it, as shown in the example below.  
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\n// Execute query to get names of all employees.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n  \"select concat(firstName, ' ', lastName) from Person\");\n\n// Iterate over the result set.\ntry (QueryCursor<List<?>> cursor = cache.query(sql) {\n  for (List<?> row : cursor)\n    System.out.println(\"personName=\" + row.get(0));\n}",
      "language": "java",
      "name": "SqlFieldsQuery"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Before specific fields can be accessed inside of `SqlQuery` or `SqlFieldsQuery`, they have to be annotated at a POJO level or defined in a `QueryEntity` so that the SQL engines becomes aware of them. Refer to [indexes](doc:indexes) documentation that covers this topic.",
  "title": "Queryable Fields Definition"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Cross-Cache Queries"
}
[/block]
The data can be queried from multiple caches as a part of single `SqlQuery` or `SqlFieldsQuery`. In this case, cache names act as schema names in conventional RDBMS like SQL queries. The name of the cache, that is used to create an instance of either `SqlQuery` or `SqlFieldsQuery`,  will be used as a default schema name and does not need to be explicitly specified. The rest of the objects, that are stored in different caches and will be queried as well, has to be prefixed with the names of their caches (additional schemas names).
[block:code]
{
  "codes": [
    {
      "code": "// In this example, suppose Person objects are stored in a \n// cache named 'personCache' and Organization objects \n// are stored in a cache named 'orgCache'.\nIgniteCache<Long, Person> personCache = ignite.cache(\"personCache\");\n\n// Select with join between Person and Organization to \n// get the names of all the employees of a specific organization.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n    \"select Person.name  \"\n        + \"from Person as p, \\\"orgCache\\\".Organization as org where \"\n        + \"p.orgId = org.id \"\n        + \"and org.name = ?\");\n\n// Execute the query and obtain the query result cursor.\ntry (QueryCursor<List<?>> cursor =  personCache.query(sql.setArgs(\"Ignite\"))) {\n    for (List<?> row : cursor)\n        System.out.println(\"Person name=\" + row.get(0));\n}",
      "language": "java",
      "name": "Cross-Cache SqlFieldsQuery"
    }
  ]
}
[/block]
In the example above an instance of `SqlFieldsQuery` is created from `personCache` which name is treated as a default schema name right after that. This is why `Person` object is accessed without explicitly specified schema name (`from Person as p`). As for `Organization` object, since it's stored in a separate cache named `orgCache`, the name of this cache must be set as a schema name explicitly in the query (`"orgCache".Organization as org`).
[block:callout]
{
  "type": "info",
  "title": "Changing Schema Name",
  "body": "If you prefer to use a schema name that is different from a cache name then you can take advantage of `CacheConfiguration.setSqlSchema(...)` method."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Distributed Joins"
}
[/block]
Ignite supports collocated and non-collocated distributed SQL joins. Moreover, if the data resides in different caches, Ignite allows for cross-cache joins as well. 
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\n// SQL join on Person and Organization.\nSqlQuery sql = new SqlQuery(Person.class,\n  \"from Person as p, \\\"orgCache\\\".Organization as org\"\n  + \"where p.orgId = org.id \"\n  + \"and lower(org.name) = lower(?)\");\n\n// Find all persons working for Ignite organization.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(\"Ignite\"))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "SqlQueryJoin"
    }
  ]
}
[/block]
Joins between `PARTITIONED` and `REPLICATED` caches always work without any limitations. 

However, if you do a join between at least two `PARTITIONED` data sets, then you must make sure that the keys you are joining on are either **collocated** or you have to enable the non-collocated joins parameter for the query. The two types of distributed joins modes are explained further below. 
[block:callout]
{
  "type": "info",
  "title": "Data Collocation",
  "body": "To learn more about data collocation concept and how to use it in practice refer to the [dedicated documentation section](doc:affinity-collocation#collocate-data-with-data)"
}
[/block]
## Distributed Collocated Joins

By default, if an SQL join has to be done across a number of Ignite caches, then all the caches have to be collocated. Otherwise, you will get an incomplete result at the end of query execution because at the join phase a node uses the data that is available only **locally**. Referring to **Picture 1.** below you will see that, first, an SQL query is sent to all the nodes (`Q`) where data, required for a join, is located. After that the query is executed right away by every node (`E(Q)`) over the local data set and, finally, the overall execution result is aggregated on the client side (`R`).  
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/2af89cf-Collocated_sql_queries.png",
        "Collocated_sql_queries.png",
        500,
        413,
        "#e2ab5b"
      ],
      "caption": "Picture 1. Collocated SQL Query"
    }
  ]
}
[/block]
## Distributed Non-Collocated Joins

Besides the fact that the affinity collocation is a powerful concept that, once set up for an application's business entities (caches), will let you execute cross-cache joins in the most optimal way by returning a complete and consistent result set, there is always a chance that you won't be able to collocate all the data. Thus, you may not be able to execute the whole range of SQL queries that are needed to satisfy your use case.

The **non-collocated** distributed joins have been designed and supported by Apache Ignite for cases when it's extremely difficult or impossible to collocate all the data but you still need to execute a number of SQL queries over non-collocated caches.
[block:callout]
{
  "type": "danger",
  "body": "Do not overuse the non-collocated distributed joins based approach in practice because the performance of this type of joins is worse then the performance of the affinity collocation based joins due to the fact that there will be much more network round-trips and data movement between the nodes to fulfill a query."
}
[/block]
When the non-collocated distributed joins setting is enabled for a specific SQL query with the `SqlQuery.setDistributedJoins(boolean)` parameter, then, the node to which the query was mapped will request for the missing data (that is not present locally) from the remote nodes by sending either broadcast or unicast requests. This is depicted on **Picture 2.** below as a potential data movement step (`D(Q)`). The potential unicast requests are only sent in cases when a join is done on a primary key (cache key) or an affinity key, since the node performing the join knows the location of the missing data. The broadcast requests are sent in all the other cases. 
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/95f09db-Non_collocated_sql_queries.png",
        "Non_collocated_sql_queries.png",
        500,
        413,
        "#dfa95b"
      ],
      "caption": "Picture 2. Non-collocated SQL Query",
      "sizing": "smart"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "success",
  "title": "",
  "body": "Neither broadcast nor unicast requests, that are sent by one node to another in order to get the missing data, are executed sequentially. The SQL engine combines all the request into batches. This batch size can be managed using `SqlQuery.setPageSize(int)` parameter."
}
[/block]
The following code snippet is provided from the [CacheQueryExample](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/datagrid/CacheQueryExample.java) included in the Ignite distribution.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<AffinityKey<Long>, Person> cache = ignite.cache(\"personCache\");\n\n// SQL clause query with join over non-collocated data.\nString joinSql =\n\t\"from Person, \\\"orgCache\\\".Organization as org \" +\n  \"where Person.orgId = org.id \" +\n  \"and lower(org.name) = lower(?)\";\n\nSqlQuery qry = new SqlQuery<AffinityKey<Long>, Person>(Person.class, joinSql).setArgs(\"ApacheIgnite\");\n\n// Enable distributed joins for the query.\nqry.setDistributedJoins(true);\n\n// Execute the query to find out employees for specified organization.\nSystem.out.println(\"Following people are 'ApacheIgnite' employees (distributed join): \", cache.query(qry).getAll());",
      "language": "java",
      "name": "SqlQueryWithDistributedJoin"
    }
  ]
}
[/block]
Refer to [the non-collocated distributed joins blog post](http://dmagda.blogspot.com/2016/08/big-change-in-apache-ignite-17-welcome.html) for more technical details.
[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
A complete example that demonstrates the usage distributed queries, covered under this documentation section, is delivered as a part of every Apache Ignite distribution and name `CacheQueryExample`. The example is [available](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/datagrid/CacheQueryExample.java) in Git Hub as well.