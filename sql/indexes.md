* [Overview](#overview)
* [Annotation Based Configuration](#annotation-based-configuration)
 * [Registering Indexed Types](#section-registering-indexed-types)
 * [Group Indexes](#section-group-indexes)
* [QueryEntity Based Configuration](#queryentity-based-configuration)
* [SkipList Based and Snapshotable Indexes](#skiplist-based-and-snapshotable-indexes)
* [Off-Heap SQL Indexes](#off-heap-sql-indexes)
* [Indexes Tradeoffs](#indexes-tradeoffs)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Presently, Apache Ignite allows defining a schema using the [annotation based](#annotation-based-configuration) or [QueryEntities based](#queryentity-based-configuration) approaches. Every specific schema is bound to an Ignite cache which name is used as the schema name in SQL queries by default.

Apache Ignite supports advanced indexing capabilities allowing you to define a single field (aka. column) or group indexes with various parameters, to manage indexes location putting them either in Java heap or off-heap spaces and so on so forth.

Indexes in Ignite are kept in a distributed fashion the same way as cache data sets. Each node that stores a specific subset of data keeps and maintains indexes corresponding to this data as well.

From this documentation page, you'll learn how to define schemas including indexes as well as queryable fields using two available approaches and how to switch between specific indexing implementations supported by data fabric. 
[block:api-header]
{
  "type": "basic",
  "title": "Annotation Based Configuration"
}
[/block]
Indexes, as well as queryable fields, can be configured from code with the usage of `@QuerySqlField` annotation. As shown in the example below, desired fields should be marked with this annotation. 
[block:code]
{
  "codes": [
    {
      "code": "public class Person implements Serializable {\n  /** Indexed field. Will be visible for SQL engine. */\n\t@QuerySqlField (index = true)\n  private long id;\n  \n  /** Queryable field. Will be visible for SQL engine. */\n  @QuerySqlField\n  private String name;\n  \n  /** Will NOT be visible for SQL engine. */\n  private int age;\n  \n  /**\n   * Indexed field sorted in descending order. \n   * Will be visible for SQL engine.\n   */\n  @QuerySqlField(index = true, descending = true)\n  private float salary;\n}",
      "language": "java"
    },
    {
      "code": "case class Person (\n  /** Indexed field. Will be visible for SQL engine. */\n  @(QuerySqlField @field)(index = true) id: Long,\n\n  /** Queryable field. Will be visible for SQL engine. */\n  @(QuerySqlField @field) name: String,\n  \n  /** Will NOT be visisble for SQL engine. */\n  age: Int\n  \n  /**\n   * Indexed field sorted in descending order. \n   * Will be visible for SQL engine.\n   */\n  @(QuerySqlField @field)(index = true, descending = true) salary: Float\n) extends Serializable {\n  ...\n}",
      "language": "scala",
      "name": "Scala"
    }
  ]
}
[/block]
Both `id` and `salary` are indexed fields. `id` field will be sorted in the ascending order (default) while `salary` in the descending order.

If you don't want to index a field but still need to use it in a SQL query, then the field has to be annotated as well omitting the `index = true` parameter. Such a field is called as a queryable field. As an example, `name` is defined as a queryable field above.

Finally, `age` is neither queryable nor indexed field and won't be accessible from SQL queries in Apache Ignite.
[block:callout]
{
  "type": "info",
  "title": "Scala Annotations",
  "body": "In Scala classes, the `@QuerySqlField` annotation must be accompanied by the `@field` annotation in order for a field to be visible for Ignite, like so:  `@(QuerySqlField @field)`. \n\nAlternatively, you can also use the `@ScalarCacheQuerySqlField` annotation from the `ignite-scalar` module which is just a type alias for the `@field` annotation."
}
[/block]
## Registering Indexed Types

After indexed and queryable fields are defined, they have to be registered in the SQL engine along with the object types they belong to.

To tell Ignite which types should be indexed, key-value pairs can be passed into `CacheConfiguration.setIndexedTypes` method as it's shown in the example below.
[block:code]
{
  "codes": [
    {
      "code": "// Preparing configuration.\nCacheConfiguration<Long, Person> ccfg = new CacheConfiguration<>();\n\n// Registering indexed type.\nccfg.setIndexedTypes(Long.class, Person.class);",
      "language": "java"
    }
  ]
}
[/block]
Note that this method accepts only pairs of types - one for key class and another for value class. Primitives are passed as boxed types.
[block:callout]
{
  "type": "info",
  "body": "In addition to all the fields marked with `@QuerySqlField` annotation, each table will have two special predefined fields: `_key` and `_val`, which represent links to whole key and value objects. This is useful, for instance, when one of them is of a primitive type and you want to filter out by its value. To do this, execute a query like `SELECT * FROM Person WHERE _key = 100`.",
  "title": "Predefined Fields"
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Since Ignite supports [Binary Marshaller](doc:binary-marshaller), there is no need to add classes of indexed types to the classpath of cluster nodes. SQL query engine is able to pick up values of indexed and queryable fields avoiding object deserialization."
}
[/block]
## Group Indexes
To set up a multi-field index that will allow accelerating queries with complex conditions, you can use `@QuerySqlField.Group` annotation. It is possible to put multiple `@QuerySqlField.Group` annotations into `orderedGroups` if you want a field to be a part of more than one group. 

For instance, in `Person` class below we have field `age` which belongs to an indexed group named `"age_salary_idx"` with group order 0 and descending sort order. Also, in the same group, we have field `salary` with group order 3 and ascending sort order. Furthermore, field `salary` itself is a single column index (there is `index = true` parameter specified in addition to `orderedGroups` declaration). Group `order` does not have to be a particular number. It is needed just to sort fields inside of a particular group. 
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
  "title": "QueryEntity Based Configuration"
}
[/block]
Indexes and queryable fields can also be configured with `org.apache.ignite.cache.QueryEntity` class which is convenient for Spring XML based configuration.

All concepts that are discussed as a part of annotation based configuration above are valid for QueryEntity based approach. Furthermore, types whose fields are configured with `@QuerySqlField` and are registered with `CacheConfiguration.setIndexedTypes` method are internally turned into query entities.

The example below shows how you can define a single field and group indexes as well as queryable fields.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"mycache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <!-- Setting indexed type's key class -->\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n              \n                <!-- Setting indexed type's value class -->\n                <property name=\"valueType\"\n                          value=\"org.apache.ignite.examples.Person\"/>\n\n                <!-- \n\t\t\t\t\t\t\t\t\t\tDefining fields that will be either indexed or queryable.\n \t\t\t\t\t\t\t\t\t\tIndexed fields are added to 'indexes' list below.\n\t\t\t\t\t\t\t\t-->\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"id\" value=\"java.lang.Long\"/>\n                        <entry key=\"name\" value=\"java.lang.String\"/>\n                        <entry key=\"salary\" value=\"java.lang.Long \"/>\n                    </map>\n                </property>\n\n                <!-- \n\t\t\t\t\t\t\t\t\t\tDefining which fields, listed above, will be treated as \n\t\t\t\t\t\t\t\t\t\tindexed fields.\n\t\t\t\t\t\t\t\t-->\n                <property name=\"indexes\">\n                    <list>\n                        <!-- Single field (aka. column) index -->\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"id\"/>\n                        </bean>\n                      \n                        <!-- Group index. -->\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg>\n                                <list>\n                                    <value>id</value>\n                                    <value>salary</value>\n                                </list>\n                            </constructor-arg>\n                            <constructor-arg value=\"SORTED\"/>\n                        </bean>\n                    </list>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "SkipList Based and Snapshotable Indexes"
}
[/block]
Ignite SQL Grid provides two indexing implementations that can be used when indexes are stored in Java heap.

The first implementation is based on a skip list data structure and is enabled by default.

The second implementation is based on a modified version of an [AVL tree with fast cloning](https://ppl.stanford.edu/papers/ppopp207-bronson.pdf). This implementation is known as `snapshotable` in Ignite and can be enabled with `CacheConfiguration.setSnapshotableIndex(...)` method.  

For off-heap mode, discussed below, Ignite provides only one indexing implementation which is a modified version of an [AVL tree with fast cloning](https://ppl.stanford.edu/papers/ppopp207-bronson.pdf).
[block:api-header]
{
  "type": "basic",
  "title": "Off-Heap SQL Indexes"
}
[/block]
Ignite supports placing indexed data in off-heap memory. This makes sense for very large datasets since keeping data in Java heap can cause high GC activity and unacceptable response times. 

By default, Ignite stores SQL Indexes on heap. Ignite will store query indexes in off-heap memory if `CacheConfiguration.setMemoryMode` is configured to one of the off-heap memory modes - `OFFHEAP_TIERED` or `OFFHEAP_VALUES`, or `CacheConfiguration.setOffHeapMaxMemory` property is set to a value >= 0.

To improve the performance of SQL queries with off-heap mode enabled, you can try to increase the value of `CacheConfiguration.setSqlOnheapRowCacheSize` property that has a default value of '10000'.
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
  "type": "info",
  "title": "Indexing Implementation",
  "body": "For off-heap mode, Ignite provides only one indexing implementation which is a modified version of an [AVL tree with fast cloning](https://ppl.stanford.edu/papers/ppopp207-bronson.pdf)."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Indexes Tradeoffs"
}
[/block]
There are multiple things you should consider when choosing indexes for your Ignite application. 

- Indexes are not free. They consume memory, also each index needs to be updated separately, thus your cache update performance can be poorer when you have more indexes set up. On top of that, the optimizer might do more mistakes by choosing a wrong index to run a query. 
[block:callout]
{
  "type": "danger",
  "title": "It is a bad strategy to index everything!"
}
[/block]
- Indexes are just sorted data structures. If you define an index for the fields `(a,b,c)` then the records will be sorted first by `a`, then by `b` and only then by `c`.
[block:callout]
{
  "type": "info",
  "body": "**| A | B | C |**\n| 1 | 2 | 3 |\n| 1 | 4 | 2 |\n| 1 | 4 | 4 |\n| 2 | 3 | 5 |\n| 2 | 4 | 4 |\n| 2 | 4 | 5 |\n\nAny condition like `a = 1 and b > 3` can be viewed as a bounded range, both bounds can be quickly looked up in in **log(N)** time, the result will be everything between.\n\nThe following conditions will be able to use the index:\n- `a = ?`\n- `a = ? and b = ?`\n- `a = ? and b = ? and c = ?`\n\nCondition `a = ? and c = ?` is no better than `a = ?` from the index point of view.\nObviously half-bounded ranges like `a > ?` can be used as well.",
  "title": "Example of Sorted Index"
}
[/block]
- Indexes on single fields are no better than group indexes on multiple fields starting with the same field (index on (a) is no better than (a,b,c)). Thus it is preferable to use group indexes.