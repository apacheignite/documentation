Ignite supports free-form SQL queries virtually without any limitations. SQL syntax is ANSI-99 compliant. You can use any SQL function, any aggregation, any grouping and Ignite will figure out where to fetch the results from.

##SQL Joins
Ignite supports distributed SQL joins. Moreover, if data resides in different caches, Ignite allows for cross-cache joins as well. 

Joins between `PARTITIONED` and `REPLICATED` caches always work without any limitations. However, if you do a join between two `PARTITIONED` data sets, then you must make sure that the keys you are joining on are **collocated**. 

##Field Queries
Instead of selecting the whole object, you can choose to select only specific fields in order to minimize network and serialization overhead. For this purpose Ignite has a concept of `fields queries`.

##Cross-Cache Queries
You can query data from multiple caches. In this case, cache names act as schema names in regular SQL. This means all caches can be referred by cache names in quotes. The cache on which the query was created acts as the default schema and does not need to be explicitly specified.
[block:callout]
{
  "type": "info",
  "title": "Performance and Usability Considerations",
  "body": "There are few common pitfalls that should be noticed when running SQL queries.\n1. If the query is using operator OR then it may use indexes not the way you would expect. For example for query `select name from Person where sex='M' and (age = 20 or age = 30)` index on field `age` will not be used even if it is obviously more selective than index on field `sex` and thus is preferable. To workaround this issue you have to rewrite the query with UNION ALL (notice that UNION without ALL will return DISTINCT rows, which will change query semantics and introduce additional performance penalty) like `select name from Person where sex='M' and age = 20 \nUNION ALL \nselect name from Person where sex='M' and age = 30`. This way indexes will be used correctly.\n2. If query contains operator IN then it has two problems: it is impossible to provide variable list of parameters (you have to specify the exact list in query like `where id in (?, ?, ?)`, but you can not write it like `where id in ?` and pass array or collection) and this query will not use index. To workaround both problems you can rewrite the query in the following way: `select p.name from Person p join table(id bigint = ?) i on p.id = i.id`. Here you can provide object array (Object[]) of any length as a parameter and the query will use index on field `id`. Note that primitive arrays (int[], long[], etc..) can not be used with this syntax, you have to pass array of boxed primitives."
}
[/block]