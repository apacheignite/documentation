* [Overview](#overview)
* [DML API](#dml-api)
* [Basic Configuration](#basic-configuration)
* [Advanced Configuration](#advanced-configuration)
* [DML Operations](#dml-operations)
 * [MERGE](dml#section-merge)
 * [INSERT](dml#section-insert)
 * [UPDATE](dml#section-update)
 * [DELETE](dml#section-delete)


  
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite SQL Grid allows not only selecting data that resides in Data Grid using SQL ANSI-99 syntax but it makes it possible to modify that data with well-known DML statements like INSERT, UPDATE or DELETE. By taking advantage of this ability, you can work with Apache Ignite In-Memory Data Fabric as with an in-memory distributed database fully relying on its SQL capabilities. 

[block:callout]
{
  "type": "info",
  "title": "SQL ANSI-99 Compliance",
  "body": "DML queries, as well as all the `SELECT` queries, are SQL ANSI-99 compliant."
}
[/block]
Since all the data is stored in Data Grid in a form of key-value entries, all the DML related operations are converted into corresponding cache key-value based commands like `cache.put(...)` or `cache.invokeAll(...)` at some stage of a DML query execution.

Let's have a deep look at how all these DML statements are implemented in practice and can be used by your application.
[block:api-header]
{
  "type": "basic",
  "title": "DML API"
}
[/block]
In general, all the DML statements can be divided into two groups. The ones that add new entries into a cache (`INSERT` and `MERGE`) and those which modify existed data (`UPDATE` and `DELETE`).

To execute these statements in Java you need to use existed `SqlFieldsQuery` API that is described in [this section](https://apacheignite.readme.io/docs/sql-queries#section-sqlfieldsqueries) in terms of its usage for `SELECT` queries. The API is used by DML operations the same way as for read-only queries except that `QueryCursor<List<?>>`, that is returned by a `SqlFieldsQuery` as a result of DML statement execution, contains a single-item `List<?>` of `long` type and that item signifies a number of cache items that were affected by the DML statement.
[block:callout]
{
  "type": "info",
  "title": "Alternative APIs",
  "body": "DML API is not limited by Java APIs only. You can connect to an Ignite cluster using ODBC or JDBC drivers and execute DML queries from their side. Learn more about additional APIs from the pages listed below:\n* [JDBC Driver](doc:jdbc-driver) \n* [ODBC Driver](doc:quering-data)"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Basic Configuration"
}
[/block]
To start using DML operations in Ignite you would need to configure queryable fields using [QueryEntity based approach](https://apacheignite.readme.io/docs/indexes#queryentity-based-configuration) or [@QuerySqlField annotations](https://apacheignite.readme.io/docs/indexes#annotation-based-configuration). Those are the fields that belong either to a cache key or value and you directly refer to them in a DML statement.

In addition to all the fields marked with @QuerySqlField annotation or defined with `QueryEntity`, there will be two special predefined fields `_key` and `_val` for every object type registered in SQL Grid. These predefined fields link to whole key and value objects stored in a cache and it's feasible to use them directly inside of DML statements as it's shown below:
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
However, it's clear that in a variety of scenarios you prefer to work with individual fields rather than with a whole object value by executing queries like the following one:
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
DML engine will be able to recreate a Person object from `firstName` and `lastName` and put it into a cache but those fields have to be defined using `QueryEntity` or `@QuerySqlField` annotation as it's shown below:
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

[block:api-header]
{
  "type": "basic",
  "title": "Advanced Configuration"
}
[/block]
##Custom Keys

If you use only predefined SQL data types for cache keys then there is no need to perform additional manipulation with DML related configuration. Those types are defined by `GridQueryProcessor#SQL_TYPES` constant and listed below.
[block:callout]
{
  "type": "info",
  "title": "Predefined SQL Data Types",
  "body": "- all the primitives and their wrappers except `char` and `Character`.\n- `String`.\n- `BigDecimal`.\n- `byte[]`.\n- `java.util.Date`, `java.sql.Date`, `java.sql.Timestamp`.\n- `java.util.UUID`."
}
[/block]
However, once you decide to introduce a custom complex key and refer to its fields from DML statements you have to:
- Define those fields in the `QueryEntity` the same way as you set fields of a value object type.
- Use the new configuration parameter `QueryEntitty.setKeyFields(..)` to distinguish key's fields from value's fields.

The example below shows how to achieve this.
[block:code]
{
  "codes": [
    {
      "code": "// Preparing cache configuration.\nCacheConfiguration cacheCfg = new CacheConfiguration<>(\"personCache\");\n\n// Creating the query entity. \nQueryEntity entity = new QueryEntity(\"CustomKey\", \"Person\");\n\n// Listing all the queryable fields.\nLinkedHashMap<String, String> flds = new LinkedHashMap<>();\n\nflds.put(\"intKeyField\", Integer.class.getName());\nflds.put(\"strKeyField\", String.class.getName());\n\nflds.put(\"firstName\", String.class.getName());\nflds.put(\"lastName\", String.class.getName());\n\nentity.setFields(flds);\n\n// Listing a subset of the fields that belong to the key.\nSet<String> keyFlds = new HashSet<>();\n\nkeyFlds.add(\"intKeyField\");\nkeyFlds.add(\"strKeyField\");\n\nentity.setKeyFields(keyFlds);\n\n// End of new settings, nothing else here is DML related\n\nentity.setIndexes(Collections.<QueryIndex>emptyList());\n\ncacheCfg.setQueryEntities(Collections.singletonList(entity));\n\nignite.createCache(cacheCfg);",
      "language": "java"
    },
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"personCache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <!-- Registering key's class. -->\n                <property name=\"keyType\" value=\"CustomKey\"/>\n              \n                <!-- Registering value's class. -->\n                <property name=\"valueType\"\n                          value=\"org.apache.ignite.examples.Person\"/>\n\n                <!-- \n                    Defining all the fields that will be accessible from DML.\n                -->\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"firstName\" value=\"java.lang.String\"/>\n                        <entry key=\"lastName\" value=\"java.lang.String\"/>\n                      \t<entry key=\"intKeyField\" value=\"java.lang.Integer\"/>\n                      \t<entry key=\"strKeyField\" value=\"java.lang.String\"/>\n                    </map>\n                </property>\n              \n                <!-- Defining the subset of key's fields -->\n                <property name=\"keyFields\">\n                    <map>\n                      \t<entry key=\"intKeyField\" value=\"java.lang.Integer\"/>\n                      \t<entry key=\"strKeyField\" value=\"java.lang.String\"/>\n                    </map>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
## HashCode Resolution and Equality Comparision for Custom Keys 

After you created a custom key and defined its fields using `QueryEntity` as it's shown above you need to take care of the hash code resolution and equality comparison .

If there's a class for the key, then a result its `hashCode` method invocation is used for its binary representation. It happens when `IgniteBinary#toBinary` is called - implicitly or explicitly.

Also, when a `BinaryIdentityResolver` is set for a binary type in configuration as shown in [this section of Binary Marshaller doc](doc:binary-marshaller#changing-default-binary-equals-and-hash-code-behav), hash code is ultimately computed by its means regardless of the way binary object was created.

But DML engine also computes hash code for binary objects created with `BinaryObjectBuilder` even when there's no `BinaryIdentityResolver` set for a binary type in configuration - it does so because in this case there's no way for a user to specify hash code to builder manually.
[block:callout]
{
  "type": "info",
  "body": "When no `BinaryIdentityResolver` is set for a binary type in configuration, and keys (or values) are built from scratch by DML engine (i.e. column values for particular fields of key and/or value are present in DML query), [BinaryArrayIdentityResolver](doc:binary-marshaller#section-binaryarrayidentityresolver) is used **both for hashing and equality comparisons**."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "DML Operations"
}
[/block]
##MERGE

`MERGE` is one of the most straightforward operations because it's translated into `cache.put(...)` and `cache.putAll(...)` operations depending on a number of rows that should be inserted or updated as a part of the `MERGE` query.

The examples below show how to update a data set with a `MERGE` command by either providing  a list of entries or injecting a result of a subquery execution. 
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

The difference between `MERGE` and `INSERT` commands is that the latter adds only those entries into a cache which keys are not there yet. 

If a single key-value pair is being added into a cache then, eventually, an `INSERT` statement will be converted into a `cache.putIfAbsent(...)` operation. In other cases, when multiple key-value pairs are inserted the DML engine creates `EntryProcessor` for each pair and uses `cache.invokeAll(...)` to propagate the data into a cache.

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

Initially, SQL engine generates and executes a `SELECT` query based on `UPDATE's `WHERE` clause and only after that modifies existing values that satisfy the clause result.

The modification is performed with the usage of `cache.invokeAll(...)` operation. Basically, it means that once the result of the `SELECT` query is ready, SQL Engine will prepare a number of `EntryProcessors` and will execute all of them using `cache.invokeAll(...)` operation. Next, while the data will be being modified using `EntryProcessors` additional checks will be performed to be sure  that nobody has interfered between the `SELECT` and the actual update.

This is simple example shows how to execute an `UPDATE` query in Apache Ignite.
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
  "title": "Inability to modify key or its fields with an UPDATE query",
  "body": "The reason behind that is that the state of the key determines internal data layout and its consistency (key's hashing and affinity, indexes integrity), so now there's no way to update a key without removing it first."
}
[/block]
##DELETE

`DELETE` statements' execution is split into two phases and similar to the execution of `UPDATE` statements. 

First, using a `SELECT` query SQL engine gathers those keys that satisfy the `WHERE` clause and have to be deleted. Next, after having all those keys at place, a number of `EntryProcessors` is created and executed with `cache.invokeAll(...)`. While the data will be being deleted this way, additional checks will be performed to be sure that nobody has interfered between the `SELECT` and the actual removal. 

This is simple example shows how to execute a `DELETE` query in Apache Ignite.
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