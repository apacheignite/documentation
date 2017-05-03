* [Overview](#section-overview)
* [CREATE INDEX](#section-create-index)
* [DROP INDEX](#section-drop-index)
[block:api-header]
{
  "title": "Overview"
}
[/block]
It is possible to create and drop SQL indexes dynamically on existing tables. Both native SQL or JDBC/ODBC drivers could be used to achieve this.
[block:api-header]
{
  "title": "CREATE INDEX"
}
[/block]
CREATE [SPATIAL] INDEX [IF NOT EXISTS] **indexName** ON **tableName** (indexColumn, ...)
indexColumn := **columnName** [ASC|DESC]
[block:api-header]
{
  "title": "DROP INDEX"
}
[/block]
TODO
[block:api-header]
{
  "title": "Using native SQL API"
}
[/block]
TODO
[block:api-header]
{
  "title": "Using JDBC driver"
}
[/block]
TODO