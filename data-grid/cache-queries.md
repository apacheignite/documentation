Ignite supports distributed cache queries allowing you to retrieve data from cache based on some user defined criteria.

##How it Works
  * The requested query is sent to each node in the cluster. 
  * Each node queries its own cache for entries that satisfy the given condition.
  * The query requester consolidates the results received from each node into a single set. 

##Types of Queries
Ignite provides a very elegant query API with support for

  * [Predicate-based Scan Queries](#scan-queries)
  * [SQL Queries](#sql-queries)
  * [Text Queries](#text-queries)
  
For SQL queries, Ignite supports in-memory indexing so all the data lookups are extremely fast. If you are caching your data in [off-heap memory](doc:off-heap-memory), then query indexes will also be cached in off-heap memory as well.

Ignite also provides support for custom indexing via `IndexingSpi` and `SpiQuery` class.
[block:api-header]
{
  "type": "basic",
  "title": "Main Abstractions"
}
[/block]
`IgniteCache` has several query methods all of which receive some sublcass of `Query` class and return `QueryCursor`.
##Query
`Query` abstract class represents an abstract paginated query to be executed on the distributed cache. You can set the page size for the returned cursor via `Query.setPageSize(...)` method (default is `1024`).

##QueryCursor
`QueryCursor` represents query result set and allows for transparent page-by-page iteration. Whenever user starts iterating over the last page, it will automatically request the next page in the background. For cases when pagination is not needed, you can use `QueryCursor.getAll()` method which will fetch the whole query result and store it in a collection.
[block:callout]
{
  "type": "info",
  "title": "Closing Cursors",
  "body": "Cursors will close automatically if you iterate to the end of the result set. If you need to stop iteration sooner, you must close() the cursor explicitly or use `AutoCloseable` syntax."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Scan Queries"
}
[/block]
Scan queries allow for querying cache in distributed form based on some user defined predicate. 
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// Find only persons earning more than 1,000.\ntry (QueryCursor cursor = cache.query(new ScanQuery((k, p) -> p.getSalary() > 1000)) {\n  for (Person p : cursor)\n    System.out.println(p.toString());\n}",
      "language": "java",
      "name": "scan"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// Find only persons earning more than 1,000.\nIgniteBiPredicate<Long, Person> filter = new IgniteByPredicate<>() {\n  @Override public boolean apply(Long key, Perons p) {\n  \treturn p.getSalary() > 1000;\n\t}\n};\n\ntry (QueryCursor cursor = cache.query(new ScanQuery(filter)) {\n  for (Person p : cursor)\n    System.out.println(p.toString());\n}",
      "language": "java",
      "name": "java7 scan"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "SQL Queries"
}
[/block]
Ignite supports free-form SQL queries virtually without any limitations. SQL syntax is ANSI-99 compliant. You can use any SQL function, any aggregation, any grouping and Ignite will figure out where to fetch the results from.

##SQL Joins
Ignite supports distributed SQL joins. Moreover, if data resides in different caches, Ignite allows for cross-cache joins as well. 

Joins between `PARTITIONED` and `REPLICATED` caches always work without any limitations. However, if you do a join between two `PARTITIONED` data sets, then you must make sure that the keys you are joining on are **collocated**. 

##Field Queries
Instead of selecting the whole object, you can choose to select only specific fields in order to minimize network and serialization overhead. For this purpose Ignite has a concept of `fields queries`.

##Cross-Cache Queries
You can query data from multiple caches. In this case, cache names act as schema names in regular SQL. This means all caches can be referred by cache names in quotes. The cache on which the query was created acts as the default schema and does not need to be explicitly specified.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\nSqlQuery sql = new SqlQuery(Person.class, \"salary > ?\");\n\n// Find only persons earning more than 1,000.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(1000))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "sql"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// SQL join on Person and Organization.\nSqlQuery sql = new SqlQuery(Person.class,\n  \"from Person, Organization \"\n  + \"where Person.orgId = Organization.id \"\n  + \"and lower(Organization.name) = lower(?)\");\n\n// Find all persons working for Ignite organization.\ntry (QueryCursor<Entry<Long, Person>> cursor = cache.query(sql.setArgs(\"Ignite\"))) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "sql join"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\nSqlFieldsQuery sql = new SqlFieldsQuery(\"select concat(firstName, ' ', lastName) from Person\");\n\n// Select concatinated first and last name for all persons.\ntry (QueryCursor<List<?>> cursor = cache.query(sql)) {\n  for (List<?> row : cursor)\n    System.out.println(\"Full name: \" + row.get(0));\n}",
      "language": "java",
      "name": "sql fields"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// Select with join between Person and Organization.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n  \"select concat(firstName, ' ', lastName), Organization.name \"\n  + \"from Person, Organization where \"\n  + \"Person.orgId = Organization.id and \"\n  + \"Person.salary > ?\");\n\n// Only find persons with salary > 1000.\ntry (QueryCursor<List<?>> cursor = cache.query(sql.setArgs(1000))) {\n  for (List<?> row : cursor)\n    System.out.println(\"personName=\" + row.get(0) + \", orgName=\" + row.get(1));\n}",
      "language": "java",
      "name": "sql fields & join"
    },
    {
      "code": "// In this example, suppose Person objects are stored in a \n// cache named 'personCache' and Organization objects \n// are stored in a cache named 'orgCache'.\n\nIgniteCache<Long, Person> personCache = ignite.cache(\"personCache\");\n\n// Select with join between Person and Organization to \n// get the names of all the employees of a specific organization.\nSqlFieldsQuery sql = new SqlFieldsQuery(\n    \"select Person.name  \"\n        + \"from Person, \\\"orgCache\\\".Organization where \"\n        + \"Person.orgId = Organization.id \"\n        + \"and Organization.name = ?\");\n\n// Execute the query and obtain the query result cursor.\ntry (QueryCursor<List<?>> cursor =  personCache.query(sql.setArgs(\"Ignite\"))) {\n    for (List<?> row : cursor)\n        System.out.println(\"Person name=\" + row);\n}",
      "language": "java",
      "name": "sql join & cross-cache"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Text Queries"
}
[/block]
Ignite also supports text-based queries based on Lucene indexing.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// Query for all people with \"Master Degree\" in their resumes.\nTextQuery txt = new TextQuery(Person.class, \"Master Degree\");\n\ntry (QueryCursor<Entry<Long, Person>> masters = cache.query(txt)) {\n  for (Entry<Long, Person> e : cursor)\n    System.out.println(e.getValue().toString());\n}",
      "language": "java",
      "name": "text query"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Query Configuration By Annotations"
}
[/block]
Indexes can be configured from code by using `@QuerySqlField` annotations. To tell Ignite which types should be indexed, key-value pairs can be passed into `CacheConfiguration.setIndexedTypes(MyKey.class, MyValue.class)` method. Note that this method accepts only pairs of types, one for key class and another for value class.
[block:code]
{
  "codes": [
    {
      "code": "public class Person implements Serializable {\n  /** Person ID (indexed). */\n  @QuerySqlField(index = true)\n  private long id;\n\n  /** Organization ID (indexed). */\n  @QuerySqlField(index = true)\n  private long orgId;\n\n  /** First name (not-indexed). */\n  @QuerySqlField\n  private String firstName;\n\n  /** Last name (not indexed). */\n  @QuerySqlField\n  private String lastName;\n\n  /** Resume text (create LUCENE-based TEXT index for this field). */\n  @QueryTextField\n  private String resume;\n\n  /** Salary (indexed). */\n  @QuerySqlField(index = true)\n  private double salary;\n  \n  ...\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Query Configuration By CacheTypeMetadata"
}
[/block]
Indexes and fields also could be configured with `org.apache.ignite.cache.CacheTypeMetadata`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n...\n <!-- Cache configuration. -->\n <property name=\"cacheConfiguration\">\n  <list>\n   <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n\t  <property name=\"name\" value=\"my_cache\"/>\n...\n     <!-- Cache types metadata. -->\n     <list>\n      <bean class=\"org.apache.ignite.cache.CacheTypeMetadata\">\n        <!-- Type to query. -->\n        <property name=\"valueType\" value=\"org.apache.ignite.examples.datagrid.store.Person\"/>\n        <!-- Fields to be queried. -->\n        <property name=\"queryFields\">\n        <map>\n         <entry key=\"id\" value=\"java.util.UUID\"/>\n         <entry key=\"orgId\" value=\"java.util.UUID\"/>\n         <entry key=\"firstName\" value=\"java.lang.String\"/>\n         <entry key=\"lastName\" value=\"java.lang.String\"/>\n         <entry key=\"resume\" value=\"java.lang.String\"/>\n         <entry key=\"salary\" value=\"double\"/>\n        </map>\n       </property>\n        <!-- Fields to index in ascending order. -->\n       <property name=\"ascendingFields\">\n        <map>\n         <entry key=\"id\" value=\"java.util.UUID\"/>\n         <entry key=\"orgId\" value=\"java.util.UUID\"/>\n         <entry key=\"salary\" value=\"double\"/>\n        </map>\n       </property>\n       <-- Fields to index as text. -->\n       <property name=\"textFields\">\n        <list>\n         <value>resume</value>\n        </list>\n      </bean>\n     </list>\n...\n  </list>\n </property>\n...\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration ccfg = new CacheConfiguration();\n....\nCollection<CacheTypeMetadata> types = new ArrayList<>();\n\nCacheTypeMetadata type = new CacheTypeMetadata();\ntype.setValueType(Person.class.getName());\n\nMap<String, Class<?>> qryFlds = type.getQueryFields();\nqryFlds.put(\"id\", UUID.class);\nqryFlds.put(\"orgId\", UUID.class);\nqryFlds.put(\"firstName\", String.class);\nqryFlds.put(\"lastName\", String.class);\nqryFlds.put(\"resume\", String.class);\nqryFlds.put(\"salary\", double.class);\n\nMap<String, Class<?>> ascFlds = type.getAscendingFields();\nascFlds.put(\"id\", UUID.class);\nascFlds.put(\"orgId\", UUID.class);\nascFlds.put(\"salary\", double.class);\n\nCollection<String> txtFlds = type.getTextFields();\ntxtFlds.add(\"resume\");\n\ntypes.add(type);\n...\nccfg.setTypeMetadata(types);\n...",
      "language": "java"
    }
  ]
}
[/block]
Note, annotations and `CacheTypeMetadata` are mutually exclusive.

For full example see `CacheQueryTypeMetadataExample`.