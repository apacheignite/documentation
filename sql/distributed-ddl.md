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
Synthax:
`CREATE [SPATIAL] INDEX [IF NOT EXISTS] indexName ON tableName (indexColumn, ...)`
`indexColumn := columnName [ASC|DESC]`
[block:code]
{
  "codes": [
    {
      "code": "CREATE INDEX idx_person_name ON Person (name)",
      "language": "sql",
      "name": "Create simple sorted index"
    }
  ]
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "CREATE INDEX idx_person_name_birth_date ON Person (name ASC, birth_date DESC)",
      "language": "sql",
      "name": "Create composite sorted index"
    }
  ]
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "CREATE SPATIAL INDEX idx_person_address ON Person (address)",
      "language": "sql",
      "name": "Create geospatial index"
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "DROP INDEX"
}
[/block]
Synthax:
`DROP INDEX [IF EXISTS] indexName`
[block:code]
{
  "codes": [
    {
      "code": "DROP INDEX idx_person_name",
      "language": "sql",
      "name": "Drop index"
    }
  ]
}
[/block]

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