Ignite now includes ODBC driver that allows you to retrieve distributed data from cache using standard SQL queries and native ODBC API.

For detailed info on ODBC please refer to [ODBC Programmer's Reference](https://msdn.microsoft.com/en-us/library/ms714177.aspx).

Apache Ignite ODBC driver implements version 3.0 of the ODBC API.
[block:api-header]
{
  "type": "basic",
  "title": "Prerequisites"
}
[/block]
Apache Ignite ODBC Driver was officially tested on:
[block:parameters]
{
  "data": {
    "0-0": "OS",
    "0-1": "Windows (XP and up, both 32-bit and 64-bit versions), \nWindows Server (2008 and up, both 32-bit and 64-bit versions)\nUbuntu (14.x and 15.x 64-bit)",
    "1-0": "C++ compiler",
    "1-1": "MS Visual C++ (10.0 and up), g++ (4.4.0 and up)",
    "2-1": "2010 and above",
    "2-0": "Visual Studio"
  },
  "cols": 2,
  "rows": 3
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Building ODBC driver"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Quering data"
}
[/block]
ODBC Driver internally uses Fields queries to retrieve data from the Apache Ignite cache. This means that by ODBC you can only access those fields that are [accessible for SQL queries](/docs/sql-queries#section-making-fields-visible-for-sql-queries).

Below you can see an example of two classes that can be queried by the ODBC Driver:
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

[block:callout]
{
  "type": "info",
  "title": "Predefined Fields",
  "body": "In addition to all the fields marked with `@QuerySqlField` annotation, each table will have two special predefined fields: `_key` and `_val`, which represent links to whole key and value objects. This is useful, for example, when one of them is a primitive and you want to filter by its value. To do this, execute a query like `SELECT * FROM Person WHERE _key = 100`."
}
[/block]
Now, lets try running a little example that will use ODBC to query some data from the Apache Ignite. First we need to modify our classes a little so we could use them in our example:
[block:code]
{
  "codes": [
    {
      "code": "/** All fields of the class will be visible in SQL. */\npublic class Person {\n  private static final AtomicLong ID_GEN = new AtomicLong();\n  \n\t@QuerySqlField\n  private long id;\n  \n  @QuerySqlField\n  public Long orgId;\n  \n  @QuerySqlField\n  private String name;\n  \n  @QuerySqlField\n  private double salary;\n  \n  public Person(Organization org, String name, double salary) {\n    id = ID_GEN.incrementAndGet();\n\n    orgId = org.id();\n\n    this.name = name;\n    this.salary = salary;\n  }\n}",
      "language": "java",
      "name": "Person"
    },
    {
      "code": "/** All fields of the class will be visible in SQL. */\npublic class Organization {\n  private static final AtomicLong ID_GEN = new AtomicLong();\n  \n  @QuerySqlField\n  private Long id;\n\n  @QuerySqlField\n  private String name;\n  \n  public Organization(String name) {\n    id = ID_GEN.incrementAndGet();\n\n    this.name = name;\n  }\n}",
      "language": "java",
      "name": "Organization"
    }
  ]
}
[/block]
Next we need to properly create and initialize caches upon which we are going to run queries:
[block:code]
{
  "codes": [
    {
      "code": "CacheConfiguration<Long, Organization> orgCacheCfg = new CacheConfiguration<>(\"Organization\");\n\norgCacheCfg.setCacheMode(CacheMode.PARTITIONED); // Default.\nogCacheCfg.setIndexedTypes(Long.class, Organization.class);\n\nCacheConfiguration<AffinityKey<Long>, Person> personCacheCfg = new CacheConfiguration<>(\"Person\");\n\npersonCacheCfg.setCacheMode(CacheMode.PARTITIONED); // Default.\npersonCacheCfg.setIndexedTypes(AffinityKey.class, Person.class);\n\n// Populate cache.\ntry (\n  IgniteCache<Long, Organization> orgCache = ignite.getOrCreateCache(orgCacheCfg);\n  IgniteCache<AffinityKey<Long>, Person> personCache = ignite.getOrCreateCache(personCacheCfg)\n) {\n  orgCache.clear();\n\n  // Organizations.\n  Organization org1 = new Organization(\"ApacheIgnite\");\n  Organization org2 = new Organization(\"Other\");\n\n  orgCache.put(org1.id(), org1);\n  orgCache.put(org2.id(), org2);\n\n  personCache.clear();\n\n  // People.\n  Person p1 = new Person(org1, \"John Doe\", 2000);\n  Person p2 = new Person(org1, \"Jane Doe\", 1000);\n  Person p3 = new Person(org2, \"John Smith\", 1000);\n  Person p4 = new Person(org2, \"Jane Smith\", 2000);\n\n  // Note that in this example we use custom affinity key for Person objects\n  // to ensure that all persons are collocated with their organizations.\n  personCache.put(p1.key(), p1);\n  personCache.put(p2.key(), p2);\n  personCache.put(p3.key(), p3);\n  personCache.put(p4.key(), p4);\n}\nfinally {\n  // Distributed cache could be removed from cluster only by #destroyCache() call.\n  ignite.destroyCache(\"Person\");\n  ignite.destroyCache(\"Organization\");\n}\n",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Joins and Collocation",
  "body": "Just like with [Cache SQL Queries](doc:cache-queries) used from `IgniteCache` API, joins on `PARTITIONED` caches will work correctly only if joined objects are stored in collocated mode. Refer to [Affinity Collocation](/docs/affinity-collocation#collocate-data-with-data) for more details."
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Cross-Cache Queries",
  "body": "Cache that the driver is connected to is treated as the default schema. To query across multiple caches, [Cross-Cache Query](/docs/cache-queries#cross-cache-queries) functionality can be used."
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Replicated vs Partitioned Caches",
  "body": "Queries on `REPLICATED` caches will run directly only on one node, while queries on `PARTITIONED` caches are distributed across all cache nodes."
}
[/block]
Finally, we can use ODBC API to run SQL query upon our data grid just like if it was ordinary database:
[block:code]
{
  "codes": [
    {
      "code": "#define BUFFER_SIZE 1024\n\nSQLHENV env;\n\n// Allocate an environment handle\nSQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &env);\n\n// We want ODBC 3 support\nSQLSetEnvAttr(env, SQL_ATTR_ODBC_VERSION, reinterpret_cast<void*>(SQL_OV_ODBC3), 0);\n\nSQLHDBC dbc;\n\n// Allocate a connection handle\nSQLAllocHandle(SQL_HANDLE_DBC, env, &dbc);\n\n// Connection string\nSQLCHAR connectStr[] = \"DRIVER={Apache Ignite};SERVER=localhost;PORT=11443;CACHE=Person;\";\nSQLSMALLINT connectStrLen = static_cast<SQLSMALLINT>(sizeof(connectStr));\n\nSQLCHAR outStr[BUFFER_SIZE] = { 0 };\nSQLSMALLINT outStrLen = static_cast<SQLSMALLINT>(sizeof(outStr));;\n\n// Connecting to ODBC server.\nSQLRETURN ret = SQLDriverConnect(dbc, NULL, connectStr, connectStrLen, outStr, outStrLen, &outStrLen, SQL_DRIVER_COMPLETE);\n\nif (!SQL_SUCCEEDED(ret))\n{\n  SQLCHAR sqlstate[7] = { 0 };\n  SQLINTEGER nativeCode;\n\n  SQLCHAR message[BUFFER_SIZE] = { 0 };\n  SQLSMALLINT reallen = 0;\n\n  SQLGetDiagRec(SQL_HANDLE_DBC, dbc, 1, sqlstate, &nativeCode, message, BUFFER_SIZE, &reallen);\n  \n  std::cerr << \"Failed to connect to Apache Ignite: \" \n            << reinterpret_cast<char*>(sqlstate) << \": \"\n            << reinterpret_cast<char*>(message) << \", \"\n            << \"Native error code: \" << nativeCode \n            << std::endl;\n\n  // Releasing allocated handles.\n  SQLFreeHandle(SQL_HANDLE_DBC, dbc);\n  SQLFreeHandle(SQL_HANDLE_ENV, env);\n  \n  return;\n}\n\nSQLHSTMT stmt;\n\n// Allocate a statement handle\nSQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\nSQLCHAR query[] = \"SELECT name, Organization.name FROM Person \"\n  \"INNER JOIN \\\"Organization\\\".Organization ON Person.orgId = Organization.id\";\nSQLSMALLINT queryLen = static_cast<SQLSMALLINT>(sizeof(queryLen));\n\nret = SQLExecDirect(stmt, query, queryLen);\n\nif (!SQL_SUCCEEDED(ret))\n{\n  SQLCHAR sqlstate[7] = { 0 };\n  SQLINTEGER nativeCode;\n\n  SQLCHAR errMsg[BUFFER_SIZE] = { 0 };\n  SQLSMALLINT reallen = 0;\n\n  SQLGetDiagRec(SQL_HANDLE_DBC, dbc, 1, sqlstate, &nativeCode, errMsg, BUFFER_SIZE, &reallen);\n  \n  std::cerr << \"Failed to connect to Apache Ignite: \" \n            << reinterpret_cast<char*>(sqlstate) << \": \"\n            << reinterpret_cast<char*>(errMsg) << \", \"\n            << \"Native error code: \" << nativeCode \n            << std::endl;\n\n  // Releasing allocated handles.\n  SQLFreeHandle(SQL_HANDLE_DBC, dbc);\n  SQLFreeHandle(SQL_HANDLE_ENV, env);\n  \n  return;\n}\n\nif (SQL_SUCCEEDED(ret))\n  PrintOdbcResultSet(stmt);\nelse\n  std::cerr << \"Failed to execute query: \" << GetOdbcErrorMessage(SQL_HANDLE_STMT, stmt) << std::endl;\n\n// Releasing statement handle.\nSQLFreeHandle(SQL_HANDLE_STMT, stmt);\n\n// Disconneting from the server.\nSQLDisconnect(dbc);\n\n// Releasing allocated handles.\nSQLFreeHandle(SQL_HANDLE_DBC, dbc);\nSQLFreeHandle(SQL_HANDLE_ENV, env);",
      "language": "cplusplus"
    }
  ]
}
[/block]