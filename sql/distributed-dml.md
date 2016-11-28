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
[block:api-header]
{
  "type": "basic",
  "title": "Special columns"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "DML Operations"
}
[/block]
##MERGE

**MERGE** is the most straightforward operation as it translates to cache **put**/**putAll** operation (depending on how many rows are listed in query, or how many rows have been returned by subquery).

SQL syntax example:
[block:code]
{
  "codes": [
    {
      "code": "merge into Person(_key, first_name, second_name) values\n  (1, \"John\", \"Smith\"),\n  (5, \"Mary\", \"Jones\")",
      "language": "sql",
      "name": "Rows List"
    },
    {
      "code": "merge into someCache.Person(_key, first_name, second_name)\n  (select _key + 1000, first_name, second_name\n   \tfrom anotherCache.Person\n   \t  where _key > 100 and _key < 200)",
      "language": "sql",
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
      "code": "insert into Person(_key, first_name, second_name) values\n  (1, \"John\", \"Smith\"),\n  (5, \"Mary\", \"Jones\")",
      "language": "sql",
      "name": "Rows list"
    },
    {
      "code": "insert into someCache.Person(_key, first_name, second_name)\n  (select _key + 1000, first_name, second_name\n   \tfrom anotherCache.Person\n   \t  where _key > 100 and _key < 200)",
      "language": "sql",
      "name": "Subquery"
    }
  ]
}
[/block]
##UPDATE

This operation updates values in cache on per field basis. First it generates and performs **SELECT** based on **UPDATE**'s **WHERE** criteria and then modifies existing values.

Actual modification is under the hood performed via cache's well known `invokeAll` operations - upon results of **SELECT**, a bunch of `EntryProcessor`s is created, and each of them modifies corresponding values checking that nobody has interfered between **SELECT** and actual update. (This particular topic will be covered below.)
[block:callout]
{
  "type": "danger",
  "title": "You can't modify key or its columns with an UPDATE query",
  "body": "The reason behind that is that the state of the key determines internal data layout and its consistency (key's hashing and affinity, indexes integrity), so now there's no way to update a key without removing it first. Probably this will change in the future."
}
[/block]
###Mind your field values priority while doing **UPDATE**