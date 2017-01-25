* [Overview](#overview)
* [DML API](#dml-api)
* [Basic Configuration](#basic-configuration)
* [Advanced Configuration](#advanced-configuration)
* [DML Operations](#dml-operations)
 * [MERGE](dml#section-merge)
 * [INSERT](dml#section-insert)
 * [UPDATE](dml#section-update)
 * [DELETE](dml#section-delete)
* [Modifications Order](#modifications-order)
* [Concurrent Modifications](#concurrent-modifications)
* [Known Limitation](#known-limitations)
* [Example](#example)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite SQL Grid not only allows selecting data from the Data Grid, using SQL ANSI-99 syntax, it makes it possible to modify that data with well-known DML statements like INSERT, UPDATE, or DELETE. By taking advantage of this ability, you can work with Apache Ignite In-Memory Data Fabric as with an in-memory distributed database fully relying on its SQL capabilities. 

[block:callout]
{
  "type": "info",
  "title": "SQL ANSI-99 Compliance",
  "body": "DML queries, as well as all the `SELECT` queries, are SQL ANSI-99 compliant."
}
[/block]
Ignite stores all the data in memory in form of key-value pairs and hence all the DML related operations are converted into corresponding cache key-value based commands like `cache.put(...)` or `cache.invokeAll(...)`. Let's take a deeper look at how the DML statements are implemented in Ignite.
[block:api-header]
{
  "type": "basic",
  "title": "DML API"
}
[/block]
In general, all the DML statements can be divided into two groups - Those that add new entries into a cache (`INSERT` and `MERGE`), and those that modify the existing data (`UPDATE` and `DELETE`).

To execute DML statements in Java, you need to use the same Ignite API that is used for `SELECT` queries -  [`SqlFieldsQuery` API](doc:sql-queries#section-sqlfieldsqueries). This API is used by DML operations the same way it is used by read-only queries, where `SqlFieldsQuery` returns `QueryCursor<List<?>>`. The only difference is that as a result of a DML statement execution, `QueryCursor<List<?>>` contains a single-item `List<?>` of `long` type that signifies the number of cache items that were affected by the DML statement, whereas as a result of a `SELECT` statement,         `QueryCursor<List<?>>`  will contain a list of items retrieved from the cache. 
[block:callout]
{
  "type": "info",
  "title": "Alternative APIs",
  "body": "DML API is not limited by Java APIs only. You can connect to an Ignite cluster using ODBC or JDBC drivers and execute DML queries from there. Learn more about additional APIs from the pages listed below:\n* [JDBC Driver](doc:jdbc-driver) \n* [ODBC Driver](doc:quering-data)"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Basic Configuration"
}
[/block]
To start using DML operations in Ignite, you would need to configure all queryable fields using [QueryEntity based approach](https://apacheignite.readme.io/docs/indexes#queryentity-based-configuration) or [@QuerySqlField annotations](https://apacheignite.readme.io/docs/indexes#annotation-based-configuration). For example:
[block:code]
{
  "codes": [
    {
      "code": "public class Person {\n  /** Field will be accessible from DML statements. */\n  @QuerySqlField\n\tprivate final String firstName;\n  \n  /** Indexed field that will be accessible from DML statements. */\n  @QuerySqlField (index = true)\n  private final String lastName;\n  \n  /** Field will NOT be accessible from DML statements. */\n  private int age;\n  \n  public Person(String firstName, String lastName) {\n  \tthis.firstName = firstName;\n    this.lastName = lastName;\n  }\n}",
      "language": "java",
      "name": "@QuerySqlField Annotations Usage"
    },
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"personCache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <!-- Registering key's class. -->\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n              \n                <!-- Registering value's class. -->\n                <property name=\"valueType\"\n                          value=\"org.apache.ignite.examples.Person\"/>\n\n                <!-- \n                    Defining fields that will be accessible from DML side\n                -->\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"firstName\" value=\"java.lang.String\"/>\n                        <entry key=\"lastName\" value=\"java.lang.String\"/>\n                    </map>\n                </property>\n              \n               <!-- \n                    Defining which fields, listed above, will be treated as \n                    indexed fields as well.\n                -->\n                <property name=\"indexes\">\n                    <list>\n                        <!-- Single field (aka. column) index -->\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"lastName\"/>\n                        </bean>\n                    </list>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml",
      "name": "QueryEntityType Usage"
    }
  ]
}
[/block]
In addition to all the fields marked with @QuerySqlField annotation or defined with `QueryEntity`, there will be two special predefined fields `_key` and `_val` for every object type registered in SQL Grid. These predefined fields provide reference to key-value entries stored in a cache and can be used directly inside of DML statements.
[block:code]
{
  "codes": [
    {
      "code": "//Preparing cache configuration.\nCacheConfiguration<Long, Person> cacheCfg = new CacheConfiguration<>\n    (\"personCache\");\n      \n//Registering indexed/queryable types.\ncacheCfg.setIndexedTypes(Long.class, Person.class);\n\n//Starting the cache.\nIgniteCache<Long, Person> cache = ignite.cache(cacheCfg);\n\n// Inserting a new key-value pair referring to prefedined `_key` and `_value`\n// fields for Person type.\ncache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, _val) VALUES(?, ?)\")\n\t.setArgs(1L, new Person(\"John\", \"Smith\")));",
      "language": "java",
      "name": "Insert Key and Value"
    }
  ]
}
[/block]
If you prefer to work with concrete fields rather than the whole object value,  you can execute a query like the one shown below:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(cacheCfg);\n\ncache.query(new SqlFieldsQuery(\n    \"INSERT INTO Person(_key, firstName, lastName) VALUES(?, ?, ?)\").\n    setArgs(1L, \"John\", \"Smith\"));",
      "language": "java",
      "name": ""
    }
  ]
}
[/block]
Note that the DML engine will be able to recreate a Person object from `firstName` and `lastName`, and put it into a cache but those fields have to be defined using `QueryEntity` or `@QuerySqlField` annotation as described above.
[block:api-header]
{
  "type": "basic",
  "title": "Advanced Configuration"
}
[/block]
##Custom Keys

If you use only predefined SQL data types for cache keys, then there is no need to perform additional manipulation with DML related configuration. Those data types are defined by `GridQueryProcessor.SQL_TYPES` constant, as listed below.
[block:callout]
{
  "type": "info",
  "title": "Predefined SQL Data Types",
  "body": "- all the primitives and their wrappers except `char` and `Character`.\n- `String`.\n- `BigDecimal`.\n- `byte[]`.\n- `java.util.Date`, `java.sql.Date`, `java.sql.Timestamp`.\n- `java.util.UUID`."
}
[/block]
However, once you decide to introduce a custom complex key and refer to its fields from DML statements, you have to:
- Define those fields in the `QueryEntity` the same way as you set fields for the value object.
- Use the new configuration parameter `QueryEntity.setKeyFields(..)` to distinguish key fields from value fields.

The example below shows how to achieve this.
[block:code]
{
  "codes": [
    {
      "code": "// Preparing cache configuration.\nCacheConfiguration cacheCfg = new CacheConfiguration<>(\"personCache\");\n\n// Creating the query entity. \nQueryEntity entity = new QueryEntity(\"CustomKey\", \"Person\");\n\n// Listing all the queryable fields.\nLinkedHashMap<String, String> flds = new LinkedHashMap<>();\n\nflds.put(\"intKeyField\", Integer.class.getName());\nflds.put(\"strKeyField\", String.class.getName());\n\nflds.put(\"firstName\", String.class.getName());\nflds.put(\"lastName\", String.class.getName());\n\nentity.setFields(flds);\n\n// Listing a subset of the fields that belong to the key.\nSet<String> keyFlds = new HashSet<>();\n\nkeyFlds.add(\"intKeyField\");\nkeyFlds.add(\"strKeyField\");\n\nentity.setKeyFields(keyFlds);\n\n// End of new settings, nothing else here is DML related\n\nentity.setIndexes(Collections.<QueryIndex>emptyList());\n\ncacheCfg.setQueryEntities(Collections.singletonList(entity));\n\nignite.createCache(cacheCfg);",
      "language": "java"
    },
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"personCache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <!-- Registering key's class. -->\n                <property name=\"keyType\" value=\"CustomKey\"/>\n              \n                <!-- Registering value's class. -->\n                <property name=\"valueType\"\n                          value=\"org.apache.ignite.examples.Person\"/>\n\n                <!-- \n                    Defining all the fields that will be accessible from DML.\n                -->\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"firstName\" value=\"java.lang.String\"/>\n                        <entry key=\"lastName\" value=\"java.lang.String\"/>\n                      \t<entry key=\"intKeyField\" value=\"java.lang.Integer\"/>\n                      \t<entry key=\"strKeyField\" value=\"java.lang.String\"/>\n                    </map>\n                </property>\n              \n                <!-- Defining the subset of key's fields -->\n                <property name=\"keyFields\">\n                    <set>\n                      \t<value>intKeyField<value/>\n                      \t<value>strKeyField<value/>\n                    </set>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
## HashCode Resolution and Equality Comparison for Custom Keys 

After creating a custom key and defining its fields using `QueryEntity`, you need to take care of the way the hash code is calculated for the key, and the way the key is compared with others.

By default, [BinaryArrayIdentityResolver](http://apacheignite.gridgain.org/v1.8/docs/binary-marshaller#handling-hash-code-generation-and-equals-execution) is used for hash code calculation and equality comparison of all the objects that are serialized and stored or transferred in Ignite. The same resolver will be used for your custom complex keys unless you change it to [BinaryFieldIdentityResolver](http://apacheignite.gridgain.org/docs/binary-marshaller#section-binary-field-identity-resolver) which is more suitable for keys used in DML statements, or switch to your custom resolver.
[block:api-header]
{
  "type": "basic",
  "title": "DML Operations"
}
[/block]
##MERGE

`MERGE` is one of the most straightforward operations because it is translated into `cache.put(...)` and `cache.putAll(...)` operations depending on the number of rows that need to be inserted or updated as part of the `MERGE` query.

The examples below show how to update the data set with a `MERGE` command by either providing  a list of entries, or injecting a result of a subquery execution. 
[block:code]
{
  "codes": [
    {
      "code": "cache.query(new SqlFieldsQuery(\"MERGE INTO Person(_key, firstName, lastName)\" + \t\"values (1, 'John', 'Smith'), (5, 'Mary', 'Jones')\"));",
      "language": "java",
      "name": "MERGE (List of Entries)"
    },
    {
      "code": "cache.query(new SqlFieldsQuery(\"MERGE INTO someCache.Person(_key, firstName, lastName) (SELECT _key + 1000, firstName, lastName \" +\n   \t\"FROM anotherCache.Person WHERE _key > ? AND _key < ?)\").setArgs(100, 200);",
      "language": "java",
      "name": "MERGE (Subquery)"
    }
  ]
}
[/block]
##INSERT

The difference between `MERGE` and `INSERT` commands is that the latter adds only those entries into a cache whose keys are not there yet.

If a single key-value pair is being added into a cache then, eventually, an `INSERT` statement will be converted into a `cache.putIfAbsent(...)` operation. In other cases, when multiple key-value pairs are inserted, the DML engine creates an `EntryProcessor` for each pair and uses `cache.invokeAll(...)` to propagate the data into a cache.

The examples below show how to insert a data set with an `INSERT` command by either providing  a list of entries or injecting a result of a subquery execution. 
[block:code]
{
  "codes": [
    {
      "code": "cache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, firstName, \" +\n         \"lastName) values (1, 'John', 'Smith'), (5, 'Mary', 'Jones')\"));",
      "language": "java",
      "name": "INSERT (List of Entries)"
    },
    {
      "code": "cache.query(new SqlFieldsQuery(\"INSERT INTO someCache.Person(_key, firstName, lastName) (SELECT _key + 1000, firstName, secondName \" +\n   \t\"FROM anotherCache.Person WHERE _key > ? AND _key < ?)\").setArgs(100, 200);",
      "language": "java",
      "name": "INSERT (Subquery)"
    }
  ]
}
[/block]
##UPDATE

This operation updates values in a cache on per field basis.

Initially, SQL engine generates and executes a `SELECT` query based on the `UPDATE WHERE` clause and only after that it modifies the existing values that satisfy the clause result.

The modification is performed via `cache.invokeAll(...)` operation. Basically, it means that once the result of the `SELECT` query is ready, SQL Engine will prepare a number of `EntryProcessors` and will execute all of them using `cache.invokeAll(...)` operation. While the data is being modified using `EntryProcessors`, additional checks are performed to make sure that nobody has interfered between the `SELECT` and the actual update.

The following example shows how to execute an `UPDATE` query in Apache Ignite.
[block:code]
{
  "codes": [
    {
      "code": "cache.query(new SqlFieldsQuery(\"UPDATE Person set lastName = ? \" +\n         \"WHERE _key >= ?\").setArgs(\"Jones\", 2L));",
      "language": "java",
      "name": "UPDATE"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "title": "Inability to modify a key or its fields with an UPDATE query",
  "body": "The reason behind that is that the state of the key determines internal data layout and its consistency (key's hashing and affinity, indexes integrity). Hence, there is no way to update a key without removing it from cache. For example, the following query:\n\nUPDATE  _key = 11 where _key = 10;\n\nmay result in the following cache operations:\n\nval = get(10);\nput(11, val);\nremove(10);"
}
[/block]
##DELETE

`DELETE` statements' execution is split into two phases and is similar to the execution of `UPDATE` statements. 

First, using a `SELECT` query, the SQL engine gathers those keys that satisfy the `WHERE` clause in the `DELETE` statement. Next, after having all those keys in place, it creates a number of `EntryProcessors`  and executes them with `cache.invokeAll(...)`. While the data is being deleted, additional checks are performed to make sure that nobody has interfered between the `SELECT` and the actual removal of the data. 

The following example shows how to execute a `DELETE` query in Apache Ignite.
[block:code]
{
  "codes": [
    {
      "code": "cache.query(new SqlFieldsQuery(\"DELETE FROM Person \" +\n         \"WHERE _key >= ?\").setArgs(2L));",
      "language": "java",
      "name": "DELETE"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Modifications Order"
}
[/block]
If a DML statement inserts/updates the whole value referring to `_val` field and at the same time tries to modify a field that belongs to `_val`, then the order in which the changes are applied is :
- The `_val` is updated/inserted first.
- The field gets updated.

The order never changes regardless of how you define it in the DML statement. For example, after the statement shown below gets executed, the final Person's value will be "Mike Smith", ignoring the fact that `_val` field appears after `firstName` in the query.
[block:code]
{
  "codes": [
    {
      "code": "cache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, firstName, _val)\" +\n           \" VALUES(?, ?, ?)\").setArgs(1L, \"Mike\", new Person(\"John\", \"Smith\")));",
      "language": "java"
    }
  ]
}
[/block]
This is similar to the execution of the query like the one below where `_val` appears before in the statement string.
[block:code]
{
  "codes": [
    {
      "code": "cache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, _val, firstName)\" +\n           \" VALUES(?, ?, ?)\").setArgs(1L, new Person(\"John\", \"Smith\"), \"Mike\"));",
      "language": "java",
      "name": "Java"
    }
  ]
}
[/block]
The order in which the changes are applied for `_val` and its fields is the same for `INSERT`, `UPDATE` and `MERGE` statements.
[block:api-header]
{
  "type": "basic",
  "title": "Concurrent Modifications"
}
[/block]
As explained above, `UPDATE` and `DELETE` statements generate `SELECT` queries internally in order to get a set of cache entries that have to be modified. The keys from the set are not locked and there is a chance that their values will be modified by other queries concurrently. A special technique is implemented by the DML engine that, first, avoids locking of keys and, second, guarantees that the values will be up-to-date at the time they will be updated by a DML statement.

Basically, the engine detects a subset of the cache entries which were modified concurrently and re-executes the `SELECT` statement limiting its scope to the modified keys only.

Let's say the following `UPDATE` statement is being executed.
[block:code]
{
  "codes": [
    {
      "code": "// Adding the cache entry.\ncache.put(1, new Person(\"John\", \"Smith\");\n          \n// Updating the entry.          \ncache.query(new SqlFieldsQuery(\"UPDATE Person set firstName = ? \" +\n         \"WHERE lastName = ?\").setArgs(\"Mike\", \"Smith\"));",
      "language": "java"
    }
  ]
}
[/block]
Before `firstName` and `lastName` are updated, the DML engine will generate the `SELECT` query to get cache entries that satisfy the`UPDATE` statement's `WHERE` clause. The statement will be the following.
[block:code]
{
  "codes": [
    {
      "code": "SELECT _key, _value, \"Mike\" from Person WHERE lastName = \"Smith\"",
      "language": "sql"
    }
  ]
}
[/block]
Right after that, the entry that was retrieved​ with the `SELECT` query can be updated concurrently.
[block:code]
{
  "codes": [
    {
      "code": "cache.put(1, new Person(\"Sarah\", \"Connor\"))",
      "language": "java"
    }
  ]
}
[/block]
The DML engine will find out that the entry with key `1` was modified at the update phase of `UPDATE` query execution. After that, it will stop the update and will re-execute a modified version the `SELECT` query in order to get latest entries' values:
[block:code]
{
  "codes": [
    {
      "code": "SELECT _key, _value, \"Mike\" from Person WHERE secondName = \"Smith\"\n    AND _key IN (SELECT * FROM TABLE(KEY long = [ 1 ]))",
      "language": "sql"
    }
  ]
}
[/block]
This query will be executed only for outdated keys. In our example, there is only one key that is `1`.

This process will repeat until the DML engine is sure at the update phase that all the entries, that are going to be updated, are up-to-date. The maximum number of attempts is `4`. Presently, there is no configuration parameter that can change this value.
[block:callout]
{
  "type": "info",
  "body": "DML engine does not re-execute the `SELECT` query for entries that are deleted concurrently​. The query is re-executed only for entries that are still in the cache."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Known Limitations"
}
[/block]
##Subqueries in WHERE clause

`SELECT` queries used in `INSERT` and `MERGE` statements as well as `SELECT` queries automatically generated by `UPDATE` and `DELETE` operations will be distributed and executed in either [collocated or non-collocated distributed modes](doc:sql-queries#distributed-joins) if needed. 

However, if there is a subquery that is executed as part of the `WHERE` clause, then it will not be executed in non-collocated distributed mode. The subquery will be executed in the collocated mode over the local data set all the times.

For example, in the following query:
[block:code]
{
  "codes": [
    {
      "code": "DELETE FROM Person WHERE _key IN\n    (SELECT personId FROM \"salary\".Salary s WHERE s.amount > 2000)",
      "language": "sql"
    }
  ]
}
[/block]
the DML engine will generate the `SELECT` query in order to get a list of entries that need to be deleted. The query will be distributed and executed across the cluster and will look like the one below:
[block:code]
{
  "codes": [
    {
      "code": "SELECT _key, _val FROM Person WHERE _key IN\n    (SELECT personId FROM \"salary\".Salary s WHERE s.amount > 2000)",
      "language": "sql"
    }
  ]
}
[/block]
However, the subquery from `IN` clause (`SELECT personId FROM "salary".Salary ...`) will not be distributed further and will be executed over the local data set present on a cluster node.

##EXPLAIN support for DML statements

Presently, `EXPLAIN` is not supported for DML operations.

One possible approach is to execute `EXPLAIN` for the `SELECT`query that is automatically generated (`UPDATE`, `DELETE`) or used (`INSERT`, `MERGE`) by DML statements. This will give you an insight on the indexes that are used when a DML operation is executed.
[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
Ignite distribution includes ready-to-run `CacheQueryDmlExample` as a part of its [sources](https://github.com/apache/ignite/blob/master/examples/src/main/java/org/apache/ignite/examples/datagrid/CacheQueryDmlExample.java). This example demonstrates the usage of all the above-mentioned DML operations.