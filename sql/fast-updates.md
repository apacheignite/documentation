[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Although usually `UPDATE` and `DELETE` operations require performing a `SELECT` in order to filter keys and values to be processed later, in some cases that may be avoided and lead to significant performance gains. Let's see why that happens and how this may be achieved.
[block:api-header]
{
  "type": "basic",
  "title": "What Makes a Fast Operation?"
}
[/block]
In the context of this document, we call a *fast update* an `UPDATE` or `DELETE` operation that:

1. Does not require performing a `SELECT`.
2. Directly maps query "as is" to a cache operation.

For above to be possible, such operation has to satisfy following criteria:

1. _(valid only for `UPDATE`)_ Set only `_val` column explicitly and no other columns - this condition provides for the lack of need to `SELECT` previous value and modify its fields.
2.