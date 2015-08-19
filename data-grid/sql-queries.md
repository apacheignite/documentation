Ignite supports free-form SQL queries virtually without any limitations. SQL syntax is ANSI-99 compliant. You can use any SQL function, any aggregation, any grouping and Ignite will figure out where to fetch the results from.

##**SQL Joins**
Ignite supports distributed SQL joins. Moreover, if data resides in different caches, Ignite allows for cross-cache joins as well. 

Joins between `PARTITIONED` and `REPLICATED` caches always work without any limitations. However, if you do a join between two `PARTITIONED` data sets, then you must make sure that the keys you are joining on are **collocated**. 

##Field Queries
Instead of selecting the whole object, you can choose to select only specific fields in order to minimize network and serialization overhead. For this purpose Ignite has a concept of `fields queries`. Also it is useful when you want to execute some aggregate query.

##Cross-Cache Queries
You can query data from multiple caches. In this case, cache names act as schema names in regular SQL. This means all caches can be referred by cache names in quotes. The cache on which the query was created acts as the default schema and does not need to be explicitly specified.
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
## Configuring SQL Indexes by Annotations
Indexes can be configured from code by using `@QuerySqlField` annotations. To tell Ignite which types should be indexed, key-value pairs can be passed into `CacheConfiguration.setIndexedTypes(MyKey.class, MyValue.class)` method. Note that this method accepts only pairs of types, one for key class and another for value class. Primitives are passed as boxed types like `CacheConfiguration.setIndexedTypes(Long.class, MyValue.class, UUID.class, MyAnotherValue.class)` (here we have 2 key-value type pairs).

In the example below we've created a simple class `Person` and annotated fields we want to  use in SQL queries with `@QuerySqlField`, fields we want to be indexed we annotated with `@QuerySqlField(index = true)` and fields we want to index as a group we annotated with `@QuerySqlField(orderedGroups={@QuerySqlField.Group(name = "age_salary_idx", order = 0)})`. 

Note that it is possible to put multiple `@QuerySqlField.Group` annotations into `orderedGroups` if you want the field to participate in more than one group index. Also note that annotating a field with `@QuerySqlField.Group` outside of `@QuerySqlField(orderedGroups={...})` will have no effect.

For example of a group index in the class below we have field `age` which participates in a group index named `"age_salary_idx"` with group order 0 and descending sort order. Also in the same group index participates field `salary` with group order 3 and ascending sort order. On top of that field `salary` itself is indexed with separate index (we have `index = true` in addition to `orderedGroups` declaration). Group `order` does not have to be any particular number, it is needed just to sort fields inside this group. 
[block:code]
{
  "codes": [
    {
      "code": "public class Person implements Serializable {\n  /** Person ID (indexed). */\n  @QuerySqlField(index = true)\n  private long id;\n\n  /** Organization ID (indexed). */\n  @QuerySqlField(index = true)\n  private long orgId;\n\n  /** First name (invisible for SQL). */\n  private String firstName;\n\n  /** Last name (not indexed but visible in SQL). */\n  @QuerySqlField\n  private String lastName;\n\n  /** Age (indexed in a group index with \"salary\"). */\n  @QuerySqlField(orderedGroups={@QuerySqlField.Group(\n    name = \"age_salary_idx\", order = 0, descending = true)})\n  private int age;\n\n  /** Salary (indexed separately and in a group index with \"age\"). */\n  @QuerySqlField(index = true, orderedGroups={@QuerySqlField.Group(\n    name = \"age_salary_idx\", order = 3)})\n  private double salary;\n  \n  ...\n}",
      "language": "java"
    }
  ]
}
[/block]
## Configuring SQL Indexes by CacheTypeMetadata
Indexes and fields also could be configured with `org.apache.ignite.cache.CacheTypeMetadata` which is convenient for XML configuration with Spring. Please refer to javadoc for details, basically it is equivalent to `@QuerySqlField` annotation.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n...\n <!-- Cache configuration. -->\n <property name=\"cacheConfiguration\">\n  <list>\n   <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n\t  <property name=\"name\" value=\"my_cache\"/>\n    <property name =\"typeMetadata\">\n     <!-- Cache types metadata. -->\n     <list>\n      <bean class=\"org.apache.ignite.cache.CacheTypeMetadata\">\n        <!-- Type to query. -->\n        <property name=\"valueType\" value=\"org.apache.ignite.examples.datagrid.store.Person\"/>\n        <!-- Fields to be queried. -->\n        <property name=\"queryFields\">\n        <map>\n         <entry key=\"id\" value=\"java.util.UUID\"/>\n         <entry key=\"orgId\" value=\"java.util.UUID\"/>\n         <entry key=\"firstName\" value=\"java.lang.String\"/>\n         <entry key=\"lastName\" value=\"java.lang.String\"/>\n         <entry key=\"resume\" value=\"java.lang.String\"/>\n         <entry key=\"salary\" value=\"double\"/>\n        </map>\n       </property>\n        <!-- Fields to index in ascending order. -->\n       <property name=\"ascendingFields\">\n        <map>\n         <entry key=\"id\" value=\"java.util.UUID\"/>\n         <entry key=\"orgId\" value=\"java.util.UUID\"/>\n         <entry key=\"salary\" value=\"double\"/>\n        </map>\n       </property>\n      </bean>\n     </list>\n...\n  </list>\n </property>\n...\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Performance and Usability Considerations",
  "body": "There are few common pitfalls that should be noticed when running SQL queries.\n\n1. If the query is using operator **OR** then it may use indexes not the way you would expect. For example for query `select name from Person where sex='M' and (age = 20 or age = 30)` index on field `age` will not be used even if it is obviously more selective than index on field `sex` and thus is preferable. To workaround this issue you have to rewrite the query with UNION ALL (notice that UNION without ALL will return DISTINCT rows, which will change query semantics and introduce additional performance penalty) like `select name from Person where sex='M' and age = 20 \nUNION ALL \nselect name from Person where sex='M' and age = 30`. This way indexes will be used correctly.\n\n2. If query contains operator **IN** then it has two problems: it is impossible to provide variable list of parameters (you have to specify the exact list in query like `where id in (?, ?, ?)`, but you can not write it like `where id in ?` and pass array or collection) and this query will not use index. To workaround both problems you can rewrite the query in the following way: `select p.name from Person p join table(id bigint = ?) i on p.id = i.id`. Here you can provide object array (Object[]) of any length as a parameter and the query will use index on field `id`. Note that primitive arrays (int[], long[], etc..) can not be used with this syntax, you have to pass array of boxed primitives."
}
[/block]