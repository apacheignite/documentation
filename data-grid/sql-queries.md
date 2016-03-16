Ignite supports free-form SQL queries virtually without any limitations. SQL syntax is ANSI-99 compliant. You can use any SQL function, any aggregation, any grouping and Ignite will figure out where to fetch the results from.

See example **SqlQuery** below.
[block:api-header]
{
  "type": "basic",
  "title": "SQL Joins"
}
[/block]
Ignite supports distributed SQL joins. Moreover, if data resides in different caches, Ignite allows for cross-cache joins as well. 

Joins between `PARTITIONED` and `REPLICATED` caches always work without any limitations. However, if you do a join between two `PARTITIONED` data sets, then you must make sure that the keys you are joining on are **collocated**. 

See example **SqlQuery JOIN** below.
[block:api-header]
{
  "type": "basic",
  "title": "Fields Queries"
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
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\nSqlQuery sql = new SqlQuery(Person.class, \"salary > ?\");\n\n// Find only persons earning more than 1,000.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(1000))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "SqlQuery"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// SQL join on Person and Organization.\nSqlQuery sql = new SqlQuery(Person.class,\n  \"from Person, Organization \"\n  + \"where Person.orgId = Organization.id \"\n  + \"and lower(Organization.name) = lower(?)\");\n\n// Find all persons working for Ignite organization.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(\"Ignite\"))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "SqlQuery JOIN"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// Select with join between Person and Organization.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n  \"select concat(firstName, ' ', lastName), Organization.name \"\n  + \"from Person, Organization where \"\n  + \"Person.orgId = Organization.id and \"\n  + \"Person.salary > ?\");\n\n// Only find persons with salary > 1000.\ntry (QueryCursor<List<?>> cursor = cache.query(sql.setArgs(1000))) {\n  for (List<?> row : cursor)\n    System.out.println(\"personName=\" + row.get(0) + \", orgName=\" + row.get(1));\n}",
      "language": "java",
      "name": "SqlFieldsQuery"
    },
    {
      "code": "// In this example, suppose Person objects are stored in a \n// cache named 'personCache' and Organization objects \n// are stored in a cache named 'orgCache'.\n\nIgniteCache<Long, Person> personCache = ignite.cache(\"personCache\");\n\n// Select with join between Person and Organization to \n// get the names of all the employees of a specific organization.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n    \"select Person.name  \"\n        + \"from Person, \\\"orgCache\\\".Organization where \"\n        + \"Person.orgId = Organization.id \"\n        + \"and Organization.name = ?\");\n\n// Execute the query and obtain the query result cursor.\ntry (QueryCursor<List<?>> cursor =  personCache.query(sql.setArgs(\"Ignite\"))) {\n    for (List<?> row : cursor)\n        System.out.println(\"Person name=\" + row);\n}",
      "language": "java",
      "name": "Cross-Cache SqlFieldsQuery"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring SQL Indexes using Annotations"
}
[/block]
Indexes can be configured from code by using `@QuerySqlField` annotations. To tell Ignite which types should be indexed, key-value pairs can be passed into `CacheConfiguration.setIndexedTypes` method, as shown in the example below. Note that this method accepts only pairs of types- one for key class and another for value class. Primitives are passed as boxed types.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<Object,Object> ccfg = new CacheConfiguration<>();\n\n// Here we are setting 3 key-value type pairs to be indexed.\nccfg.setIndexedTypes(\n  MyKey.class, MyValue.class,\n\tLong.class, MyOtherValue.class,\n  UUID.class, String.class\n);",
      "language": "java"
    }
  ]
}
[/block]
## Making Fields Visible for SQL Queries
To make fields accessible for SQL queries you have to annotate them with `@QuerySqlField`. Field `age` will not be accessible from SQL. Note that none of these fields are indexed. 
[block:code]
{
  "codes": [
    {
      "code": "public class Person implements Serializable {\n  /** Will be visible in SQL. */\n\t@QuerySqlField\n  private long id;\n  \n  /** Will be visible in SQL. */\n  @QuerySqlField\n  private String name;\n  \n  /** Will NOT be visible in SQL. */\n  private int age;\n}",
      "language": "java"
    },
    {
      "code": "case class Person (\n  /** Will be visible in SQL. */\n  @(QuerySqlField @field) id: Long,\n\n  /** Will be visible in SQL. */\n  @(QuerySqlField @field) name: String,\n  \n  /** Will NOT be visisble in SQL. */\n  age: Int\n) extends Serializable {\n  ...\n}",
      "language": "scala",
      "name": "Scala"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Predefined Fields",
  "body": "In addition to all the fields marked with `@QuerySqlField` annotation, each table will have two special predefined fields: `_key` and `_val`, which represent links to whole key and value objects. This is useful, for example, when one of them is a primitive and you want to filter by its value. To do this, execute a query like `SELECT * FROM Person WHERE _key = 100`."
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Scala Annotations",
  "body": "In Scala classes, the `@QuerySqlField` annotation must be accompanied by the `@field` annotation in order for a field to be visible for Ignite, like so:  `@(QuerySqlField @field)`. \n\nAlternatively, you can also use the `@ScalarCacheQuerySqlField` annotation from the `ignite-scalar` module which is just a type alias for the `@field` annotation."
}
[/block]
## Single Column Indexes
To make fields not only accessible by SQL but also speedup queries you can index field values. To create a single column index you can annotate field with `@QuerySqlField(index = true)`.  
[block:code]
{
  "codes": [
    {
      "code": "public class Person implements Serializable {\n  /** Will be indexed in ascending order. */\n\t@QuerySqlField(index = true)\n  private long id;\n  \n  /** Will be visible in SQL, but not indexed. */\n  @QuerySqlField\n  private String name;\n  \n  /** Will be indexed in descending order. */\n  @QuerySqlField(index = true, descending = true)\n  private int age;\n}",
      "language": "java"
    },
    {
      "code": "case class Person (\n  /** Will be indexed in ascending order. */\n  @(QuerySqlField @field)(index = true) id: Long,\n  \n  /** Will be visible in SQL, but not indexed. */\n  @(QuerySqlField @field) name: String,\n  \n  /** Will be indexed in descending order. */\n  @(QuerySqlField @field)(index = true, descending = true) age: Int\n) extends Serializable {\n  ...\n}",
      "language": "scala"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Scala Annotations",
  "body": "In Scala classes, the indexed `@QuerySqlField` annotation should look like so: `@(QuerySqlField @field)(index = true)`."
}
[/block]
## Group Indexes
To have a multi-field index to speedup queries with complex conditions, you can use `@QuerySqlField.Group` annotation. It is possible to put multiple `@QuerySqlField.Group` annotations into `orderedGroups` if you want the field to participate in more than one group index. 

For example of a group index in the class below we have field `age` which participates in a group index named `"age_salary_idx"` with group order 0 and descending sort order. Also in the same group index participates field `salary` with group order 3 and ascending sort order. On top of that field `salary` itself is indexed with single column index (we have `index = true` in addition to `orderedGroups` declaration). Group `order` does not have to be any particular number, it is needed just to sort fields inside of this group. 
[block:code]
{
  "codes": [
    {
      "code": "public class Person implements Serializable {\n  /** Indexed in a group index with \"salary\". */\n  @QuerySqlField(orderedGroups={@QuerySqlField.Group(\n    name = \"age_salary_idx\", order = 0, descending = true)})\n  private int age;\n\n  /** Indexed separately and in a group index with \"age\". */\n  @QuerySqlField(index = true, orderedGroups={@QuerySqlField.Group(\n    name = \"age_salary_idx\", order = 3)})\n  private double salary;\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "body": "Note that annotating a field with `@QuerySqlField.Group` outside of `@QuerySqlField(orderedGroups={...})` will have no effect."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring SQL Indexes using QueryEntity"
}
[/block]
Indexes and fields also could be configured with `org.apache.ignite.cache.QueryEntity` which is convenient for XML configuration with Spring. Please refer to javadoc for details. It is equivalent to using `@QuerySqlField` annotation because class annotations are converted to query entities internally.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"mycache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"org.apache.ignite.examples.Person\"/>\n\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"id\" value=\"java.lang.Long\"/>\n                        <entry key=\"name\" value=\"java.lang.String\"/>\n                        <entry key=\"age\" value=\"java.lang.Integer\"/>\n                    </map>\n                </property>\n\n                <property name=\"indexes\">\n                    <list>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"id\"/>\n                        </bean>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"age\"/>\n                        </bean>\n                    </list>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "How SQL Queries Work"
}
[/block]
There are two main ways of how query can be processed in Ignite:

1. If you execute the query against `REPLICATED` cache, Ignite assumes that all the data is available locally and will run a simple local SQL query in the H2 database engine. The same will happen for `LOCAL` caches.

2. If you execute the query against `PARTITIONED` cache, it work the following way: the query will be parsed and split into multiple map queries and a single reduce query. Then all the map queries are executed on all data nodes of participating caches, providing results to the reducing node, which will in turn run the reduce query over these intermediate results.
[block:callout]
{
  "type": "warning",
  "title": "",
  "body": "Note that Ignite queries work only with data that is already loaded to a cache and do not trigger read-through from the CacheStore. You need to preload the cache with data before running a query against it."
}
[/block]

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
        "https://www.filepicker.io/api/file/X2drTdAyQBiOqjPYpbDg",
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
  "title": "Off-heap SQL Indexes"
}
[/block]
Ignite supports placing index data to off-heap memory. This makes sense for very large datasets when keeping data on heap causes high GC activity and unacceptable response times. 

SQL Indexes will reside on heap when property `CacheConfiguration.setOffHeapMaxMemory` is set to `-1`, otherwise off-heap indexes are always used. Note that it is the only property to enable or disable off-heap indexing, while for example  `CacheConfiguration.setMemoryMode` does not have any effect on indexing.

To improve performance of SQL queries with off-heap enabled, you can try to increase value of property `CacheConfiguration.setSqlOnheapRowCacheSize` which can be low by default `10 000`.
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<Object, Object> ccfg = new CacheConfiguration<>();\n\n// Set unlimited off-heap memory for cache and enable off-heap indexes.\nccfg.setOffHeapMaxMemory(0); \n\n// Cache entries will be placed on heap and can be evited to off-heap.\nccfg.setMemoryMode(ONHEAP_TIERED);\nccfg.setEvictionPolicy(new RandomEvictionPolicy(100_000));\n\n// Increase size of SQL on-heap row cache for off-heap indexes.\nccfg.setSqlOnheapRowCacheSize(100_000);",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "body": "Note that SQL queries are not supported if cache memory mode is set to OFFHEAP_VALUES."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Choosing Indexes"
}
[/block]
There are multiple things you should consider when choosing indexes for your Ignite application. 

- Indexes are not free. They consume memory, also each index needs to be updated separately, thus your cache update performance can be lower if you have more indexes. On top of that optimizer can do more mistakes choosing wrong index to run query. 
[block:callout]
{
  "type": "danger",
  "title": "It is a bad strategy to index everything!",
  "body": ""
}
[/block]
- Indexes are just sorted data structures. If you define an index on the fields (a,b,c) , the records are sorted first on a, then b, then c.
[block:callout]
{
  "type": "info",
  "body": "**| A | B | C |**\n| 1 | 2 | 3 |\n| 1 | 4 | 2 |\n| 1 | 4 | 4 |\n| 2 | 3 | 5 |\n| 2 | 4 | 4 |\n| 2 | 4 | 5 |\n\nAny condition like `a = 1 and b > 3` can be viewed as a bounded range, both bounds can be quickly looked up in index in **log(N)** time, the result will be everything between.\n\nThe following conditions will be able to use the index:\n- `a = ?`\n- `a = ? and b = ?`\n- `a = ? and b = ? and c = ?`\n\nCondition `a = ? and c = ?` is no better than `a = ?` from the index point of view.\nObviously half-bounded ranges like `a > ?` can be used as well.",
  "title": "Example of Sorted Index"
}
[/block]
- Indexes on a single fields are no better than group indexes on multiple fields starting with the same field (index on (a) is no better than (a,b,c)). Thus it is preferable to use group indexes.
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