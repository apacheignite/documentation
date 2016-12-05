Since 1.8.0, Ignite is capable not only of querying data from cache, but also to modify it. Supported operations include **MERGE** (a.k.a. upsert), **INSERT**, **UPDATE**, and **DELETE**, and each of them maps to a specific cache operation.

Let's have a closer look at basic concepts and how operations work.
[block:api-header]
{
  "type": "basic",
  "title": "Basic Concepts"
}
[/block]
Four DML operations mentioned above can be split in two small groups: the ones that put new items to cache (that would be **MERGE** and **INSERT**) and those that modify existing items (**UPDATE** and **DELETE**). Let's discuss the former group first.

##Put new items to cache
Both **MERGE** and **INSERT** put new key-value pairs to cache, and their syntax is nearly identical as you will see soon. The difference between them is semantic.

**MERGE** puts given pair(s) to cache without considering current presence of keys in the cache, which is identical to, say, MySQL's `INSERT ... ON DUPLICATE KEY UPDATE`.

In its turn, **INSERT** puts only pairs for which the keys are not in cache yet - it's identical to semantic of **INSERT** in conventional RDBMSes.

Both **MERGE** and **INSERT** may work in two modes - rows list based and subquery based. In first case, the user explicitly lists field values in tuples each of which then is converted to key-value pair and put to cache. In second case, first SQL SELECT is done, and then each row of its results serves as base tuple for new key-value pair.

As long as SQL in case of Ignite is merely an interface to query or manipulate cache data, in the end all DML operations boil down to modifying key-value pairs that reside in cache. Thus, as all columns in Ignite's tables correspond either to key or to value, when a tuple (which is a "new row") is processed, key and value get instantiated and get their fields set based on what has been passed in tuple. After all tuples are well-formed, cache modifying operations are performed.

##Modify existing cache items
**UPDATE** and **DELETE** are the operations responsible for this, with former updating values in cache (per field or replacing them completely), and the latter removing entries from the cache. They both include **WHERE** clause that allows the user to specify which rows exactly must be modified.

##Special columns
As you may know, each Ignite's SQL table has two special columns - those are `_key` and `_val`. They correspond to complete key and value respectively, although of course the table most likely has also the columns corresponding to particular fields of key or value.
Suppose we have such class (we'll use it in examples below as well):
[block:code]
{
  "codes": [
    {
      "code": "public class Person {\n\tprivate final String firstName;\n  \n  private final String secondName;\n  \n  public double salary;\n  \n  public Person(String firstName, String secondName) {\n  \tthis.firstName = firstName;\n    this.secondName = secondName;\n  }\n}",
      "language": "java",
      "name": "Model Person class"
    }
  ]
}
[/block]
Therefore, the simplest way to put an item into cache via DML is as follows:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"INSERT INTO Person(_key, _val) VALUES(?, ?)\")\n         .setArgs(1L, new Person(\"John\", \"Smith\")));",
      "language": "java",
      "name": "Insert explicit value as _val"
    }
  ]
}
[/block]
However, DML engine is capable of building either cache key or value from individual field values - therefore, we could write the above query like this:
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
In this case, DML engine will build `Person` object or its binary version by itself and put it to cache.

## Field values override
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
  "title": "Configuration changes to enable DML operations"
}
[/block]
If your caches use only primitive/SQL types as keys **OR** if you do not use `BinaryMarshaller`, then you basically nothing to worry about - you can use DML operations right out of the box without any configuration changes.
[block:callout]
{
  "type": "info",
  "title": "SQL data types",
  "body": "They include:\n- primitives and their wrappers,\n- `Strings`,\n- `BigDecimal`s,\n- **non wrapped** byte arrays - `byte[]`,\n- `java.util.Date`, `java.sql.Date`, `java.sql.Timestamp`,\n- and `java.util.UUID`."
}
[/block]
**But**, if you use complex keys and binary marshaller, and your keys and values are classless on server (and, probably, client) nodes, i.e. are described as a `QueryEntity` with explicitly stated fields and their types, then you must tell Ignite which columns correspond to keys and which belong to values. New configuration param `QueryEntitty#keyFields` is responsible for that. As stated above, it is in no way mandatory and is overall honored only if the key is of non SQL type.
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
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.query(new SqlFieldsQuery(\"MERGE INTO Person(_key, firstName, \" +\n         \"secondName) values (1, 'John', 'Smith'), (5, 'Mary', 'Jones')\"));",
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
      "code": "IgniteCache<Long, Person> cache = ignite.cache(\"personCache\");\n\ncache.put(1L, new Person(\"John\", \"Smith\");\ncache.put(2L, new Person(\"Sarah\", \"Jones\");\n\ncache.query(new SqlFieldsQuery(\"UPDATE Person set salary = ? \" +\n         \"WHERE _key >= ?\").setArgs(5000, 2L)); // Sorry Mike...",
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
###Mind your field value overrides while doing **UPDATE**
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
will set the value for key `1L` to **Mike Jones** because new value for `_val` column is present (**Sarah Jones**) and it's taken as basis for new value for the key. It also can be positioned anywhere in updated columns list compared to values of individual fields.

##DELETE
Winner in straightforwardness: simply filters keys for which condition in **WHERE** holds true and removes them, performing **SELECT** to find those keys - just like with **UPDATE**, that **SELECT** may be distributed two-step or local. (More on this will follow below.).

Inner implementation of cache modifications is also quite similar to **UPDATE** - after **SELECT**, `EntryProcessor`s are created for each found key which are then run via `invokeAll` and then