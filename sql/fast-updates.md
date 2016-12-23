[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
This document is about deep optimization technique used by Ignite to speed up DML statements of particular kind.

Although usually `UPDATE` and `DELETE` operations require performing a `SELECT` in order to filter keys and values to be processed later, in some cases that may be avoided and may lead to significant performance gains by effectively translating DML statements into cache operations without any additional overhead. Let's see why that happens and how this may be achieved.
[block:api-header]
{
  "type": "basic",
  "title": "Why Query?"
}
[/block]
The reasons why `UPDATE` and `DELETE` usually require performing a `SELECT` are as follows:

1. A complex filter is present in `WHERE` condition - we really have to perform some work to figure out entries that will be affected by DML statement as query does not tell us as it is what has to be affected.
2. _(valid only for `UPDATE`)_ Statement contains expressions - even if our `WHERE` is simple and directly points to the cache entry to be modified, and even more so if it's not, new column values could be computed by some functions or expressions. We have to run a `SELECT` to evaluate all of those in this case.
3. _(valid only for `UPDATE`)_ Distinct fields of cache entry value are updated - we have to retrieve current value for a given key first, then modify it thus making new value object, then put it back to cache.
[block:api-header]
{
  "type": "basic",
  "title": "What Makes a Fast Operation?"
}
[/block]
In accordance with the previous section, within the context of this document we call a *fast update* an `UPDATE` or `DELETE` operation that:

1. Does not require performing a `SELECT`.
2. Updates single cache entry.
3. Directly maps query "as is" to a cache operation.

For above to be possible, such operation has to satisfy following criteria:

1. Filter only by key equality, and, *optionally*, value equality - no other conditions must be present.
2. Filtering arguments for both key and value (if it's present) must be either present explicitly in query string as a constant, or must be taken from query params - in other words, `WHERE` part of a fast operation can't contain any function calls or expressions that need to be evaluated, as either would take making a `SELECT`, and that's what a fast operation strives to avoid.
3. _(valid only for `UPDATE`)_ Set only `_val` column explicitly and no other columns - this condition provides for the lack of need to `SELECT` previous value and modify its fields.
[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
Let's see an example:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Integer, Integer> intCache = getIntCache();\n\nintCache.query(new SqlFieldsQuery(\"UPDATE Person SET _val = ?3 WHERE _key = ?1 and _val = ?2\").setArgs(7, 1, 2));",
      "language": "java"
    }
  ]
}
[/block]
As you may see, this `UPDATE` statement
- explicitly tells us which cache entry has to be updated (by specifying `_key` that does not need to be computed but rather is provided directly),
- sets `_val` explicitly again with directly provided value thus freeing us from necessity to `SELECT` and modify existing value,
- imposts additional condition onto `_val` - cache contents will be modified only if current value for given key is equal to that of given parameter.

And thus it is effectively executed as following cache operation - please mind param indexes in previous code snippet to see for yourself that `7` corresponds to key while `1` and `2` correspond to expected old and new values for that key respectively.
[block:code]
{
  "codes": [
    {
      "code": "intCache.replace(7, 1, 2);",
      "language": "java"
    }
  ]
}
[/block]
As said before, `_val` bound is optional, so query snippet could look like this
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Integer, Integer> intCache = getIntCache();\n\nintCache.query(new SqlFieldsQuery(\"UPDATE Person SET _val = ?2 WHERE \" +\n \"_key = ?1\").setArgs(7, 2));",
      "language": "java"
    }
  ]
}
[/block]
And it would the be directly mapped to
[block:code]
{
  "codes": [
    {
      "code": "intCache.replace(7, 2);",
      "language": "java"
    }
  ]
}
[/block]
For simplicity, above example use primitive types both for key and value, although of course either could be an object.

`DELETE` statements follow the same optimization rules - the main thing to consider is the kind of filtering:
- `_key` and, optionally, `_val` columns,
- equality comparison for both key and value,
- no expressions in condition and new value for `_val` (in case of `UPDATE`).