* [Configuring SQL Indexes by Annotations](#configuring-sql-indexes-by-annotations)
* [Making Fields Visible for SQL Queries](#section-making-fields-visible-for-sql-queries)
* [Single Column Indexes](#section-single-column-indexes)
* [Group Indexes](#section-group-indexes)
* [Configuring SQL Indexes Using QueryEntity](#configuring-sql-indexes-using-queryentity)
[block:api-header]
{
  "type": "basic",
  "title": "Configuring SQL Indexes by Annotations"
}
[/block]
Indexes can be configured from code by using `@QuerySqlField` annotations. To tell Ignite which types should be indexed, key-value pairs can be passed into `CacheConfiguration.setIndexedTypes` method, as shown in the example below. Note that this method accepts only pairs of types - one for key class and another for value class. Primitives are passed as boxed types.
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
  "body": "In addition to all the fields marked with `@QuerySqlField` annotation, each table will have two special predefined fields: `_key` and `_val`, which represent links to whole key and value objects. This is useful, for example, when one of them is a primitive and you want to filter by its value. To do this, execute a query like `SELECT * FROM Person WHERE _key = 100`.",
  "title": "Predefined Fields"
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
To make fields not only accessible by SQL but also speed up queries you can index field values. To create a single column index you can annotate a field with `@QuerySqlField(index = true)`.
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
  "type": "warning",
  "body": "Note that annotating a field with `@QuerySqlField.Group` outside of `@QuerySqlField(orderedGroups={...})` will have no effect."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring SQL Indexes Using QueryEntity"
}
[/block]
Indexes and fields can also be configured with `org.apache.ignite.cache.QueryEntity` which is convenient for XML configuration with Spring. It is equivalent to using `@QuerySqlField` annotation because class annotations are converted to query entities internally.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"mycache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"org.apache.ignite.examples.Person\"/>\n\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"id\" value=\"java.lang.Long\"/>\n                        <entry key=\"name\" value=\"java.lang.String\"/>\n                        <entry key=\"age\" value=\"java.lang.Integer\"/>\n                        <entry key=\"salary\" value=\"java.lang.Long \"/>\n                    </map>\n                </property>\n\n                <property name=\"indexes\">\n                    <list>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"id\"/>\n                        </bean>\n                        <!-- Group index. -->\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg>\n                                <list>\n                                    <value>age</value>\n                                    <value>salary</value>\n                                </list>\n                            </constructor-arg>\n                            <constructor-arg value=\"SORTED\"/>\n                        </bean>\n                    </list>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]