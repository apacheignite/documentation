Starting from Ignite 1.8, DML is supported in SQL queries. This means DML can now be used in ODBC as well. On this you can find details.

[block:code]
{
  "codes": [
    {
      "code": "/** All fields of the class will be visible in SQL. */\npublic class Person {\n\t@QuerySqlField\n  private long id;\n  \n  @QuerySqlField\n  public Long orgId;\n  \n  @QuerySqlField\n  private String name;\n  \n  @QuerySqlField\n  private double salary;\n}",
      "language": "java",
      "name": "Person"
    },
    {
      "code": "/** All fields of the class will be visible in SQL. */\npublic class Organization {\n  @QuerySqlField\n  private Long id;\n\n  @QuerySqlField\n  private String name;\n}",
      "language": "java",
      "name": "Organization"
    }
  ]
}
[/block]