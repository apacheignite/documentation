* [Overview](#overview)
* [DML API](#dml-api)
* [Basic Configuration](#basic-configuration)
* [Advanced Configuration](#advanced-configuration)
* [DML Operations](#dml-operations)
 *[MERGE](dml#section-merge)


  
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

But DML engine also computes hash code for binary objects created with `BinaryObjectBuilder` **even when there's no `BinaryIdentityResolver` set for a binary type in configuration** - it does so because in this case there's no way for a user to specify hash code to builder manually.
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

The examples below show how to update the data set with a `MERGE` command by either providing  a list of entries or injecting a result of a subquery execution. 
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

The difference between `MERGE` and `INSERT` commands is that the latter adds only those entries into a cache which keys are not there yet.  `INSERT` statement supports data addition in a form of a list of entries as well as a result of the subquery.
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

[block:api-header]
{
  "type": "basic",
  "title": "Values' Fields Overriding Specificities"
}
[/block]
##Field values override
When `_key` (or `_val`) column value is given in DML query and that query also includes individual values from key (or value) columns correspondingly, first `_key` (or `_val`) column value is taken, and then individual field values are overridden, if any. For example, if we issue the following query,
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, firstName, \" + \t\t\t\t\t\t\t\t \"_val) VALUES(?, ?, ?)\").setArgs(1L, \"Mike\", new Person(\"John\",  \t\t\t\t\t\t\t \"Smith\")));",
      "language": "java"
    }
  ]
}
[/block]
then DML engine will take `Person` named **John Smith** and passed as a query argument as the basis and set value of `firstName` field to **Mike**, and resulting `Person` will be **Mike Smith**, even though `_val` column in the query is mentioned _after_ `firstName`. This behavior holds for all DML operations that build keys and/or values to put to cache - namely, **MERGE**, **INSERT**, and **UPDATE**.

###Field value overrides with **UPDATE**
As stated in section [field values override](#section-field-values-override), **UPDATE** also honors value of `_val` column while processing rows. But, in contrary with **MERGE** and **INSERT**, **UPDATE** always deals only with existing entries - and that's why there's some value to `_val` column whose fields are modified by values for other columns (if any). For example, if we do this:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.put(1L, new Person(\"John\", \"Smith\");\n\ncache.query(new SqlFieldsQuery(\"UPDATE Person set firstName = ? \" +\n         \"WHERE _key = 1\").setArgs(\"Mike\"));",
      "language": "java",
      "name": "UPDATE field values override example 1"
    }
  ]
}
[/block]
then resulting person will be **Mike Smith**, because **UPDATE** takes existing `_val` (**John Smith**) and updates its `firstName` field. And the following query
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.put(1L, new Person(\"John\", \"Smith\");\n\ncache.query(new SqlFieldsQuery(\"UPDATE Person set firstName = ?, \" +\n         \"_val = ? WHERE _key = 1\").setArgs(\"Mike\", new Person(\"Sarah\",\n         \"Jones\")));",
      "language": "java",
      "name": "UPDATE field values override example 2"
    }
  ]
}
[/block]
will set the value for key `1L` to **Mike Jones** because new value for `_val` column is present (**Sarah Jones**) and it's taken as basis for the new key. It also can be positioned anywhere in updated columns list compared to values of individual fields.
[block:api-header]
{
  "type": "basic",
  "title": "UPDATE and DELETE Concurrency"
}
[/block]
As explained above, **UPDATE** and **DELETE** first perform **SELECT** and then modify filtered items - this may be viewed as a map-reduce of some sort by itself. No data is blocked after **SELECT**, and therefore it's possible that someone else's operation slips between **SELECT** and actual data modification thus probably rendering **SELECT** results invalid.

For example, if someone does a cache `put` while data is filtered *but not yet processed* by **UPDATE**, then value is not the same as it was during **SELECT**, and we don't know if it's still right to modify filtered item as long as **WHERE**'s criteria could have stopped holding true for it. To check for such situations, DML engine's entry processors check that current value stored in cache entry is equal (Java's `equals` wise) to that present in cache during **SELECT**.

If some keys were modified concurrently, all of them are collected, and a **query re-run** is performed automatically. A re-run rewrites original query appending to its **WHERE** additional criteria **which limits its scope with the keys that failed to be updated previously.

Say, if we had such query
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.put(1L, new Person(\"John\", \"Smith\");\n\ncache.query(new SqlFieldsQuery(\"UPDATE Person set firstName = ? \" +\n         \"WHERE secondName = ?\").setArgs(\"Mike\", \"Smith\"));",
      "language": "java"
    }
  ]
}
[/block]
then it would generate such **SELECT** under the hood...
[block:code]
{
  "codes": [
    {
      "code": "SELECT _key, _value, \"Mike\" from Person WHERE secondName = \"Smith\"",
      "language": "sql"
    }
  ]
}
[/block]
And, if between running that **SELECT** and data modification someone had managed to interfere with this:
[block:code]
{
  "codes": [
    {
      "code": "ignite.cache(\"personCache\").put(1L, new Person(\"Sarah\", \"Jones\"))",
      "language": "java"
    }
  ]
}
[/block]
then we would find **Sarah Jones** and not **John Smith** when DML engine's entry processor attempted to modify entry's value by setting it to **Mike Smith**. On a re-run, **SELECT** would look like this:
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
Note the part after **AND**: this query preserves original condition and protects any unrelated entries from being changed by limiting its own scope with only particular set of keys (ones for which it failed before).

If this query returns nothing for some key, then original criteria does not hold for that key anymore (`1` in our example), and no modification attempts are made for that key further.

However, if some key gets **SELECT**ed again, DML engine attempts to modify its value again. And, of course, it could fail once more. Should that happen, a re-run is done again (of course, excluding those keys from previous re-run that have been processed successfully), and the process repeats, shrinking set of target keys with each attempt.
[block:callout]
{
  "type": "info",
  "body": "Should at some point entry disappear from the cache at all, no further attempts to do anything with it are made - in other words, a re-run is made only for the keys that are still present in cache, gone entries are ignored."
}
[/block]
Currently Ignite shall make at most **four** consecutive attempts to execute DML query as explained above - that is an initial attempt and at most three re-runs, if needed. Number of attempts is currently "hard coded" and probably some configuration param will be added in the future to set that, or maybe some set of predefined policies to control that behavior.
[block:api-header]
{
  "type": "basic",
  "title": "Two-step and Local Operations"
}
[/block]
**INSERT** and **MERGE** queries are capable of retrieving data for new cache keys and values from subqueries which are **SELECT**s declared by the user. **UPDATE** and **DELETE** queries  filter items that should be affected by them by running a **SELECT** query which, in case of **UPDATE**, also computes new values for updated columns. And, as explained [here](doc:distributed-queries) and [here](doc:local-queries), there are **distributed** and **local** queries. That said, **SELECT** for any DML operation (if any) may either be distributed (aka *two-step*) and local.

##Local operations
**SELECT** will run locally in precisely the same cases as stated in [Local Queries](doc:local-queries) doc - you can either:

- force this flag in `SqlFieldsQuery`,
- or execute a query on `LOCAL` cache,
- or execute it against a `REPLICATED` cache on a node where the cache is deployed.

Still, **actual data modification affects whole cache**, distributed or not, just as it would when doing an ordinary cache `put`, `replace`, or whatever.

##Two-step operations
These run **SELECT**s in map-reduce manner as explained in [Distributed Queries](doc:sql-queries) doc. **Actual data modification still affects whole cache.** 
[block:callout]
{
  "type": "success",
  "body": "All settings related with distrubuted joins, etc., are propagated from original `SqlFieldsQuery` to new one when doing a two-step **SELECT**, so that the user has more fine-grained control over its execution.",
  "title": "Fine query settings are honored by DML too"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Known Limitations"
}
[/block]
##Subqueries in WHERE are not distributed
Although subqueries for **INSERT** and **MERGE** may be distributed, as well as **SELECT**s generated by **UPDATE** and **DELETE**, if a **WHERE** clause of such query contains a subquery, **it will not be distributed**. Example:
[block:code]
{
  "codes": [
    {
      "code": "DELETE FROM Person WHERE _key IN (SELECT _key FROM \"otherCache\".Person p WHERE p.salary > 2000)",
      "language": "sql"
    }
  ]
}
[/block]
In this case, subquery in **WHERE** (the one that deals with `otherCache` inside braces after **IN**) won't be distributed.

##UPDATE is not supported for key or its fields
As mentioned in section [UPDATE](#section-update), updates to key or its fields are not supported for now. This might change in the course of work on transactional DML operations.

##No EXPLAIN for DML operations
Ignite performs DML operations in a very different way than a conventional RDBMS does. Therefore using H2's **EXPLAIN** output with DML operations as it's done with **SELECT**s does not look like a good solution. One possible approach is just to **EXPLAIN** the **SELECT** generated/used by DML operations (because in many cases there's one) - this will give insight on which indexes are used while executing DML operations. Probably this will be implemented in near future.