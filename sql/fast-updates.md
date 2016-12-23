[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
This document is about deep optimization technique used by Ignite to speed up particular class of DML statements.

Although usually `UPDATE` and `DELETE` operations require performing a `SELECT` in order to filter keys and values to be processed later, in some cases that may be avoided and may lead to significant performance gains by effectively translating DML statements into cache operations without any additional overhead. Let's see why that happens and how this may be achieved.
[block:api-header]
{
  "type": "basic",
  "title": "Why Query?"
}
[/block]
The reasons why `UPDATE` and `DELETE` usually require performing a `SELECT` are as follows:

1. A complex filter is present in `WHERE` condition - we really have to perform some work to filter entries that will be affected by DML statement.
2. _(valid only for `UPDATE`)_ Distinct fields of cache entry value are updated - we have to retrieve current value for a given key first, then modify it thus making new value object, then put it back to cache.
3. _(valid only for `UPDATE`)_ Statement contains expressions - even if our `WHERE` is simple, and even more so if it's not, new column values could be computed by some functions or expressions. We have to run a `SELECT` to evaluate all of those in this case.
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

Let's see

[block:code]
{
  "codes": [
    {
      "code": "UPDATE Person SET _val = ? WHERE _key = ? and _val = ?",
      "language": "sql"
    }
  ]
}
[/block]