* [Overview](#overview)
* [Main Abstractions](#main-abstractions)
 * [Query](#section-query)
 * [QueryCursor](#section-querycursor) 
* [Scan Queries](#scan-queries)
* [SQL Queries](#sql-queries)
* [Text Queries](#text-queries)
* [Query Configuration by Annotations](#query-configuration-by-annotations)
* [Query Configuration using QueryEntity ](#query-configuration-using-queryentity)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite supports a very elegant query API with support for Predicate-based Scan Queries, SQL Queries (ANSI-99 compliant), and Text Queries. For SQL queries ignites supports in-memory indexing, so all the data lookups are extremely fast. If you are caching your data in [off-heap memory](doc:off-heap-memory), then query indexes will also be cached in off-heap memory as well.

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
  "body": "Cursors will close automatically if you call method `QueryCursor.getAll()`. If you iterating over the cursor in a for loop or explicitly getting `Iterator`, you must close() the cursor explicitly or use `AutoCloseable` syntax."
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
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"mycache\");\n\n// Find only persons earning more than 1,000.\nIgniteBiPredicate<Long, Person> filter = new IgniteBiPredicate<>() {\n  @Override public boolean apply(Long key, Perons p) {\n  \treturn p.getSalary() > 1000;\n\t}\n};\n\ntry (QueryCursor cursor = cache.query(new ScanQuery(filter)) {\n  for (Person p : cursor)\n    System.out.println(p.toString());\n}",
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
Ignite SQL queries are covered in a separate section [SQL Queries](doc:sql-queries).
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
  "title": "Query Configuration by Annotations"
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
  "title": "Query Configuration using QueryEntity"
}
[/block]
Indexes and fields also could be configured with `org.apache.ignite.cache.QueryEntity` which is convenient for XML configuration with Spring. Please refer to javadoc for details. It is equivalent to using `@QuerySqlField` annotation because class annotations are converted to query entities internally.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"mycache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"org.apache.ignite.examples.Person\"/>\n\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"id\" value=\"java.lang.Long\"/>\n                        <entry key=\"orgId\" value=\"java.lang.Long\"/>\n                        <entry key=\"firstName\" value=\"java.lang.String\"/>\n                        <entry key=\"lastName\" value=\"java.lang.String\"/>\n                        <entry key=\"resume\" value=\"java.lang.String\"/>\n                        <entry key=\"salary\" value=\"java.lang.Double\"/>\n                    </map>\n                </property>\n\n                <property name=\"indexes\">\n                    <list>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"id\"/>\n                        </bean>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"orgId\"/>\n                        </bean>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"salary\"/>\n                        </bean>\n                    </list>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration<Long, Person> cacheCfg = new CacheConfiguration<>();\n...\ncacheCfg.setName(\"mycache\");\n\n// Setting up query entity.\nQueryEntity queryEntity = new QueryEntity();\n\nqueryEntity.setKeyType(Long.class.getName());\nqueryEntity.setValueType(Person.class.getName());\n\n// Listing query fields.\nLinkedHashMap<String, String> fields = new LinkedHashMap();\n\nfields.put(\"id\", Long.class.getName());\nfields.put(\"orgId\", Long.class.getName());\nfields.put(\"firstName\", String.class.getName());\nfields.put(\"lastName\", String.class.getName());\nfields.put(\"resume\", String.class.getName());\nfields.put(\"salary\", Double.class.getName());\n\nqueryEntity.setFields(fields);\n\n// Listing indexes.\nCollection<QueryIndex> indexes = new ArrayList<>(3);\n\nindexes.add(new QueryIndex(\"id\"));\nindexes.add(new QueryIndex(\"orgId\"));\nindexes.add(new QueryIndex(\"salary\"));\n\nqueryEntity.setIndexes(indexes);\n...\ncacheCfg.setQueryEntities(Arrays.asList(queryEntity));\n...",
      "language": "java"
    }
  ]
}
[/block]