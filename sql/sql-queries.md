* [Overview](#overview)
* [How SQL Queries Work](#how-sql-queries-work)
* [SQL Joins](#sql-joins)
* [Field Queries](#field-queries)
* [Cross-Cache Queries](#cross-cache-queries)
* [Distributed Joins](#distributed-joins)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite supports free-form SQL queries truly without any limitations. The SQL syntax is ANSI-99 compliant which literally means that you can use any kind of SQL functions, aggregations, groupings or joins, defined by the specification, as a part of an SQL query.

Furthermore, the queries are fully distributed. The SQL engine is capable not only of mapping a query to specific nodes and reducing their responses into a final result set but it's also able to join data sets stored in different caches and even on various nodes. Lastly, it worth to mention, that the engine does this in a fault-tolerant fashion guaranteeing that you will never get an incomplete or wrong result due to a topology change event, that happens when a new node joins the cluster or an old one leaves it.
[block:api-header]
{
  "type": "basic",
  "title": "How SQL Queries Work"
}
[/block]

There are two main ways of how query can be processed in Ignite:

1. If you execute the query against `REPLICATED` cache, Ignite assumes that all the data is available locally and will run a simple local SQL query in the H2 database engine. The same will happen for `LOCAL` caches.

2. If you execute the query against `PARTITIONED` cache, it will be executed ub the following way: the query will be parsed and split into multiple map queries and a single reduce query. Then all the map queries are executed on all data nodes of participating caches, providing results to the reducing node, which will in turn run the reduce query over these intermediate results.
[block:api-header]
{
  "type": "basic",
  "title": "SQL Joins"
}
[/block]
Ignite supports collocated and non-collocated distributed SQL joins. Moreover, if the data resides in different caches, Ignite allows for cross-cache joins as well. 

Joins between `PARTITIONED` and `REPLICATED` caches always work without any limitations. However, if you do a join between two `PARTITIONED` data sets, then you must make sure that the keys you are joining on are either **collocated** or you have enabled the non-collocated joins parameter for the query. 

See example **SqlQuery JOIN** below.
[block:api-header]
{
  "type": "basic",
  "title": "Field Queries"
}
[/block]
Instead of selecting the whole object, you can choose to select only specific fields in order to minimize network and serialization overhead. For this purpose Ignite has a concept of `fields queries`. Also it is useful when you want to execute some aggregate query.

See example **SqlFieldsQuery** below.
[block:api-header]
{
  "type": "basic",
  "title": "Cross-Cache Queries"
}
[/block]

You can query data from multiple caches. In this case, cache names act as schema names in regular SQL. This means all caches can be referred by cache names in quotes. The cache on which the query was created acts as the default schema and does not need to be explicitly specified.

See example **Cross-Cache SqlFieldsQuery**.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\nSqlQuery sql = new SqlQuery(Person.class, \"salary > ?\");\n\n// Find all persons earning more than 1,000.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(1000))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "SqlQuery"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\n// SQL join on Person and Organization.\nSqlQuery sql = new SqlQuery(Person.class,\n  \"from Person as p, \\\"orgCache\\\".Organization as org\"\n  + \"where p.orgId = org.id \"\n  + \"and lower(org.name) = lower(?)\");\n\n// Find all persons working for Ignite organization.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(\"Ignite\"))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "SqlQuery JOIN"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\n// Execute query to get names of all employees.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n  \"select concat(firstName, ' ', lastName) from Person\");\n\n// Iterate over the result set.\ntry (QueryCursor<List<?>> cursor = cache.query(sql) {\n  for (List<?> row : cursor)\n    System.out.println(\"personName=\" + row.get(0));\n}",
      "language": "java",
      "name": "SqlFieldsQuery"
    },
    {
      "code": "// In this example, suppose Person objects are stored in a \n// cache named 'personCache' and Organization objects \n// are stored in a cache named 'orgCache'.\nIgniteCache<Long, Person> personCache = ignite.cache(\"personCache\");\n\n// Select with join between Person and Organization to \n// get the names of all the employees of a specific organization.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n    \"select Person.name  \"\n        + \"from Person as p, \\\"orgCache\\\".Organization as org where \"\n        + \"p.orgId = org.id \"\n        + \"and org.name = ?\");\n\n// Execute the query and obtain the query result cursor.\ntry (QueryCursor<List<?>> cursor =  personCache.query(sql.setArgs(\"Ignite\"))) {\n    for (List<?> row : cursor)\n        System.out.println(\"Person name=\" + row.get(0));\n}",
      "language": "java",
      "name": "Cross-Cache SqlFieldsQuery"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Distributed Joins"
}
[/block]
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
  "title": "Using EXPLAIN"
}
[/block]
Ignite supports "EXPLAIN ..." syntax in SQL queries and reading execution plans is a main way to analyze query performance in Ignite. Note that plan cursor will contain multiple rows: the last one will contain query for reducing node, others are for map nodes.
[block:code]
{
  "codes": [
    {
      "code": "SqlFieldsQuery sql = new SqlFieldsQuery(\n  \"explain select name from Person where age = ?\").setArgs(26); \n\nSystem.out.println(cache.query(sql).getAll());",
      "language": "java"
    }
  ]
}
[/block]
The execution plan itself is generated by H2 as described here: 
http://www.h2database.com/html/performance.html#explain_plan
[block:api-header]
{
  "type": "basic",
  "title": "Using H2 Debug Console"
}
[/block]
When developing with Ignite sometimes it is useful to check if your tables and indexes look  correctly or run some local queries against embedded in node H2 database. For that purpose Ignite has an ability to start H2 Console. To do that you can start a local node with `IGNITE_H2_DEBUG_CONSOLE` system property or environment variable set to `true`. The console will be opened in your browser. Probably you will need to click `Refresh` button on the Console because it can be opened before database objects initialized. 
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/OsddL8lfTOSLKqZWaTlI_Screen%20Shot%202015-08-24%20at%207.06.36%20PM.png",
        "Screen Shot 2015-08-24 at 7.06.36 PM.png",
        "1394",
        "1018",
        "#945642",
        ""
      ],
      "caption": ""
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Performance and Usability Considerations"
}
[/block]
There are few common pitfalls that should be noticed when running SQL queries.

1. If the query is using operator **OR** then it may use indexes not the way you would expect. For example for query `select name from Person where sex='M' and (age = 20 or age = 30)` index on field `age` will not be used even if it is obviously more selective than index on field `sex` and thus is preferable. To workaround this issue you have to rewrite the query with UNION ALL (notice that UNION without ALL will return DISTINCT rows, which will change query semantics and introduce additional performance penalty) like `select name from Person where sex='M' and age = 20 
UNION ALL 
select name from Person where sex='M' and age = 30`. This way indexes will be used correctly.

2. If query contains operator **IN** then it has two problems: it is impossible to provide variable list of parameters (you have to specify the exact list in query like `where id in (?, ?, ?)`, but you can not write it like `where id in ?` and pass array or collection) and this query will not use index. To workaround both problems you can rewrite the query in the following way: `select p.name from Person p join table(id bigint = ?) i on p.id = i.id`. Here you can provide object array (Object[]) of any length as a parameter and the query will use index on field `id`. Note that primitive arrays (int[], long[], etc..) can not be used with this syntax, you have to pass array of boxed primitives.