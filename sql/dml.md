* [Overview](#overview)
  
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

Let's have a deep look on how all these DML statements are implemented in practice and can be used by your application.
[block:api-header]
{
  "type": "basic",
  "title": "DML Statements"
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
##Configuration

To start using DML operations in Ignite you need to configure queryable fields using [QueryEntity based approach](https://apacheignite.readme.io/docs/indexes#queryentity-based-configuration) or [@QuerySqlField annotations](https://apacheignite.readme.io/docs/indexes#annotation-based-configuration). Those are the fields that belong either to a cache key or value and you directly refer to them in a DML statement.

In addition to all the fields marked with @QuerySqlField annotation or defined with `QueryEntity`, there will be two special predefined fields `_key` and `_val` for every object type registered in SQL Grid. These predefined fields link to whole key and value objects stored in a cache and it's feasible to use them directly inside of DML statements as it's shown below:
[block:code]
{
  "codes": [
    {
      "code": "//Preparing cache configuration.\nCacheConfiguration<Long, Person> cfg = new CacheConfiguration<>(\"personCache\");\n      \n//Registering indexed/queryable types.\ncfg.setIndexedTypes(Long.class, Person.class);\n\n//Starting the cache.\nIgniteCache<Long, Person> cache = ignite.cache(cfg);\n\n// Inserting a new key-value pair referring to prefedined `_key` and `_value`\n// fields for Person type.\ncache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, _val) VALUES(?, ?)\")\n\t.setArgs(1L, new Person(\"John\", \"Smith\")));",
      "language": "java",
      "name": "Insert Key and Value"
    }
  ]
}
[/block]
However, there is a variety of scenarios when you prefer to work with individual fields rather than with a whole object value by executing queries like the following one:
 
If your caches use only primitive/SQL types as keys **OR** if you do not use `BinaryMarshaller`, then you basically have nothing to worry about - you can use DML operations right out of the box without any configuration changes.
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, firstName, \" + \t\t\t\t\t\t\t\t \"secondName) VALUES(?, ?, ?)\").setArgs(1L, \"John\", \"Smith\"));",
      "language": "java",
      "name": "Build value from individual fields"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "SQL data types",
  "body": "They include:\n- primitives and their wrappers - except `char` and `Character`,\n- `String`s,\n- `BigDecimal`s,\n- **non wrapped** byte arrays - `byte[]`,\n- `java.util.Date`, `java.sql.Date`, `java.sql.Timestamp`,\n- and `java.util.UUID`.\n\nPlease refer to `GridQueryProcessor#SQL_TYPES` constant for the list of types."
}
[/block]
**But**, if you use complex keys and binary marshaller, and your keys and/or values are classless on server (and, probably, client) nodes, i.e. are described as a `QueryEntity` with explicitly stated fields and their types, then you must tell Ignite which columns correspond to keys and which belong to values. New configuration param `QueryEntitty#keyFields` is responsible for that. It's a `Set<String>` which contains names of fields that corresponds to **key**.

As stated above, this new field is in no way mandatory if you don't use DML and is overall honored only if the key is of non SQL type. Here's how to configure Ignite for DML in code and via XML - it's pretty simple and the only thing that needs to be updated is `QueryEntity` section:

[block:code]
{
  "codes": [
    {
      "code": "// Don't mind cacheConfig method - let's just assume it creates new cache config\n// which has all params set besides QueryEntities.\nCacheConfiguration cacheCfg = cacheConfig(\"cache\");\n\nQueryEntity entity = new QueryEntity(\"Key\", \"Person\");\n\nLinkedHashMap<String, String> flds = new LinkedHashMap<>();\n\nflds.put(\"intKeyField\", Integer.class.getName());\nflds.put(\"strKeyField\", String.class.getName());\n\nflds.put(\"firstName\", String.class.getName());\nflds.put(\"secondName\", String.class.getName());\n\nentity.setFields(flds);\n\n// The following block is what you need to add to make DML work -\n// all the rest you probably had set even before DML\n\n// Look, the same names can be seen in 'flds' above\nSet<String> keyFlds = new HashSet<>();\nkeyFlds.add(\"intKeyField\");\nkeyFlds.add(\"strKeyField\");\n\nentity.setKeyFields(keyFlds);\n\n// End of new settings, nothing else here is DML related\n\nentity.setIndexes(Collections.<QueryIndex>emptyList());\n\ncacheCfg.setQueryEntities(Collections.singletonList(entity));\n\nignite.createCache(cacheCfg);",
      "language": "java"
    },
    {
      "code": "<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xsi:schemaLocation=\"http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\">\n\n<bean id=\"ignite.cfg\"\n      class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\t<property name=\"cacheConfiguration\">\n  \t\t<list>\n  \t\t\t<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n  \t\t\t\t<!-- The rest of configuration is omitted for clarity -->\n  \t\t\t\t<property name=\"queryEntities\">\n  \t\t\t\t\t<list>\n                        <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                            <property name=\"keyType\" value=\"Key\"/>\n                            <property name=\"valueType\" value=\"Person\"/>\n\n                            <property name=\"fields\">\n                                <map>\n                                    <entry key=\"intKeyField\" value=\"java.lang.Integer\"/>\n                                    <entry key=\"strKeyField\" value=\"java.lang.String\"/>\n\n                                    <entry key=\"firstName\" value=\"java.lang.String\"/>\n                                    <entry key=\"secondName\" value=\"java.lang.String\"/>\n                                </map>\n                            </property>\n\n                            <!-- Note that fields with such names\n                            are present in fields list above -->\n                            <property name=\"keyFields\">\n                                <list>\n                                    <value>intKeyField</value>\n                                    <value>strKeyField</value>\n                                </list>\n                            </property>\n                        </bean>\n  \t\t\t\t\t</list>\n  \t\t\t\t</property>\n  \t\t\t</bean>\n  \t\t</list>\n\t</property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "public class Person {\n  @QuerySqlField\n\tprivate final String firstName;\n  \n  @QuerySqlField\n  private final String secondName;\n  \n  public Person(String firstName, String secondName) {\n  \tthis.firstName = firstName;\n    this.secondName = secondName;\n  }\n}",
      "language": "java",
      "name": "Person"
    }
  ]
}
[/block]
##Put new items to cache
Both **MERGE** and **INSERT** put new key-value pairs to cache, and their syntax is nearly identical as you will see soon. The difference between them is semantic.

**MERGE** puts given pair(s) to cache without considering current presence of keys in the cache, which is identical to, say, MySQL's `INSERT ... ON DUPLICATE KEY UPDATE`.

In its turn, **INSERT** puts only pairs for which the keys are not in cache yet - it's identical to semantic of **INSERT** in conventional RDBMSes.

Both **MERGE** and **INSERT** may work in two modes - rows list based and subquery based. In first case, the user explicitly lists field values in tuples each of which then is converted to key-value pair and put to cache. In second case, first SQL SELECT is done, and then each row of its results serves as base tuple for new key-value pair.

As long as SQL in case of Ignite is merely an interface to query or manipulate cache data, in the end all DML operations boil down to modifying key-value pairs that reside in cache. Thus, as all columns in Ignite's tables correspond either to key or to value, when a tuple (which is a "new row") is processed, key and value get instantiated and get their fields set based on what has been passed in tuple. After all tuples are well-formed, cache modifying operations are performed.

##Modify existing cache items
**UPDATE** and **DELETE** are the operations responsible for this, with former updating values in cache (per field or replacing them completely), and the latter removing entries from the cache. They both include **WHERE** clause that allows the user to specify which rows exactly must be modified.

##Special columns

However, DML engine is capable of building either cache key or value from individual field values - therefore, we could write the above query like this:

In this case, DML engine will build `Person` object or its binary version by itself and put it to cache.

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
[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "DML Operations"
}
[/block]
##MERGE

**MERGE** is the most straightforward operation (except probably for **DELETE**) as it translates to cache **put**/**putAll** operation (depending on how many rows are listed in query, or how many rows have been returned by subquery).

SQL syntax example:
[block:code]
{
  "codes": [
    {
      "code": "\nIgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"MERGE INTO Person(_key, firstName, \" +\n         \"secondName) values (1, 'John', 'Smith'), (5, 'Mary', 'Jones')\"));",
      "language": "java",
      "name": "Rows List"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"MERGE INTO someCache.Person(_key, firstName, secondName) (SELECT _key + 1000, firstName, secondName \" +\n   \t\"FROM anotherCache.Person WHERE _key > ? AND _key < ?)\").setArgs(100, 200);",
      "language": "java",
      "name": "Subquery"
    }
  ]
}
[/block]
As you may see, there's two modes to **MERGE** - one that takes tuples corresponding to new rows and another that takes data to put to cache from subquery.

##INSERT

**INSERT** is semantically different from **MERGE**, but still quite similar: it puts to cache only values whose keys are not present in cache. Besides that, they are nearly identical from user perspective. **INSERT** also supports rows based and subquery based operations:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, firstName, \" +\n         \"secondName) values (1, 'John', 'Smith'), (5, 'Mary', 'Jones')\"));",
      "language": "java",
      "name": "Rows list"
    },
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"INSERT INTO someCache.Person(_key, firstName, secondName) (SELECT _key + 1000, firstName, secondName \" +\n   \t\"FROM anotherCache.Person WHERE _key > ? AND _key < ?)\").setArgs(100, 200);",
      "language": "java",
      "name": "Subquery"
    }
  ]
}
[/block]
##UPDATE

This operation updates values in cache on per field basis. First it generates and performs **SELECT** based on **UPDATE**'s **WHERE** criteria and then modifies existing values.

Actual modification is under the hood performed via cache's well known `invokeAll` operations - upon results of **SELECT**, a bunch of `EntryProcessor`s is created, and each of them modifies corresponding values checking that nobody has interfered between **SELECT** and actual update. (This particular topic will be covered below.)

SQL syntax example:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.put(1L, new Person(\"John\", \"Smith\");\ncache.put(2L, new Person(\"Sarah\", \"Jones\");\n\ncache.query(new SqlFieldsQuery(\"UPDATE Person set salary = ? \" +\n         \"WHERE _key >= ?\").setArgs(5000, 2L)); // Sorry John...",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "title": "You can't modify key or its columns with an UPDATE query",
  "body": "The reason behind that is that the state of the key determines internal data layout and its consistency (key's hashing and affinity, indexes integrity), so now there's no way to update a key without removing it first. Probably this will change in the future."
}
[/block]
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

##DELETE
Winner in straightforwardness: simply filters keys for which **WHERE** condition holds true and removes them, performing **SELECT** to find those keys - just like with **UPDATE**, that **SELECT** may be distributed two-step or local. (More on this will follow below.). Example:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.put(1L, new Person(\"John\", \"Smith\");\ncache.put(2L, new Person(\"Sarah\", \"Jones\");\n\ncache.query(new SqlFieldsQuery(\"DELETE FROM Person \" +\n         \"WHERE _key >= ?\").setArgs(2L)); // Sorry Sarah...",
      "language": "java"
    }
  ]
}
[/block]
Inner implementation of cache modifications is also quite similar to **UPDATE** - after **SELECT**, `EntryProcessor`s are created for each found key which are then run via `invokeAll` and then updates cache entry is nobody got ahead of it (to determine that, the value present in cache at the time of **SELECT** is compared for equality with that being there at the time of `EntryProcessor` execution).

Behavior in case of concurrent modification of cache entries will be described further below.
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
  "title": "Hashing of Non Primitive Binary Keys"
}
[/block]
When building a new (non primitive binary) key on **MERGE** or **INSERT**, newly built object must have a hash code for correct data layout and keys distribution among nodes. If there's a class for the key, then result of invoking its `hashCode` method is used for its binary representation. It happens when `IgniteBinary#toBinary` is called - implicitly or explicitly.

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