* [Overview](#section-overview)
* [CREATE INDEX](#section-create-index)
* [DROP INDEX](#section-drop-index)
* [Apache Ignite SQL API](#section-apache-ignite-sql-api)
* [JDBC driver](#section-jdbc-driver)
[block:api-header]
{
  "title": "Overview"
}
[/block]
Starting with Apache Ignite 2.0 it is possible to use Data Definition Language (DDL) statements for the sake of SQL indexes creation and removal in runtime. Both native Apache Ignite SQL APIs, as well as JDBC and ODBC drivers, can be used for a SQL schema modifications.
[block:callout]
{
  "type": "success",
  "title": "Full-fledged DDL Support",
  "body": "In the future Apache Ignite releases, you can expect to see support for additional widely used DDL statements."
}
[/block]

[block:api-header]
{
  "title": "CREATE INDEX"
}
[/block]
Syntax:
`CREATE [SPATIAL] INDEX [IF NOT EXISTS] indexName ON tableName (indexColumn, ...)`
`indexColumn := columnName [ASC|DESC]` where `tableName` is a name of the type stored in a distirbuted cache.

Here is how a simple sorted index can be created:​
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
To create a composite index use a command like the one below:
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
Add `SPATIAL` keyword to define a geo-spatial index:
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
Syntax:
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
  "title": "Apache Ignite SQL API"
}
[/block]
DDL commands could be executed via `SqlFieldsQuery` class as it's shown below:
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<PersonKey, Person> cache = ignite.cache(\"Person\");\n\nSqlFieldsQuery query = new SqlFieldsQuery(\n    \"CREATE INDEX idx_person_name ON Person (name)\");\n\ncache.query(query).getAll();",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "JDBC driver"
}
[/block]
The example below demonstrates how to create an index with Apache Ignite JDBC driver.
[block:code]
{
  "codes": [
    {
      "code": "Class.forName(\"org.apache.ignite.IgniteJdbcDriver\");\n\nConnection conn = DriverManager.getConnection(\n    \"jdbc:ignite:cfg://file:///etc/config/ignite-jdbc.xml\");\n\ntry (Statement stmt = conn.createStatement()) {\n    stmt.execute(\"CREATE INDEX idx_person_name ON Person (name)\");\n}",
      "language": "java"
    }
  ]
}
[/block]