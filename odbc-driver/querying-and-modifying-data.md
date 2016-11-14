* [Preparing node](#preparing-node)
* [Connecting to the node](#connecting-to-the-node)
* [Inserting records](#inserting-records)
* [Updating records](#updating-records)
* [Deleting records](#deleting-records)

ODBC Driver internally uses Fields queries to retrieve data from the Apache Ignite cache. This means that by ODBC you can only access those fields that are [accessible for SQL queries](/docs/sql-queries#section-making-fields-visible-for-sql-queries).

Starting from Ignite 1.8, DML is supported in SQL queries. This means DML can now be used in ODBC as well. Here you can find details and some sample code on how to do that.

Also, you can refer to [ODBC Example](https://github.com/apache/ignite/tree/master/modules/platforms/cpp/examples/odbc-example) in Ignite Git repository for the example code.
[block:api-header]
{
  "type": "basic",
  "title": "Preparing node"
}
[/block]
With DML you can now insert, modify and delete records using SLQ alone, which means you can now manipulate data on Ignite writing virtually zero lines of Java code. Let me give you an example. Consider the following simple node configuration:
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\n        http://www.springframework.org/schema/util\n        http://www.springframework.org/schema/util/spring-util.xsd\">\n  <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <!-- Enabling ODBC. -->\n    <property name=\"odbcConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.OdbcConfiguration\"/>\n    </property>\n\n    <!-- Configuring cache. -->\n    <property name=\"cacheConfiguration\">\n      <list>\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Person\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Person\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"name\" value=\"java.lang.String\"/>\n                    <entry key=\"salary\" value=\"java.lang.Double\"/>\n                  </map>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n        \n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Organization\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Organization\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"name\" value=\"java.lang.String\"/>\n                  </map>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n      </list>\n    </property>\n  </bean>\n</beans>\n",
      "language": "xml",
      "name": "XML"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Note odbcConfiguration property.",
  "body": "Remember, you need explicitly enable ODBC on the node you going to connect to."
}
[/block]
As you can see, we have here two caches: `Person` and `Organization`, that contain types with matching names and some fields - `name` and `salary` for `Person` and `name` for `Organization`. Not much, but enough for the demonstration.
[block:callout]
{
  "type": "info",
  "title": "DiscoverySpi was not configured for this node.",
  "body": "You can read about how to configure it on [Cluster Configuration](doc:cluster-config) page."
}
[/block]
Of course you can still configure node from the Java code, as well as use `@QuerySqlField` annotations. Refer to [Cache Queries](doc:cache-queries) for details.
[block:api-header]
{
  "type": "basic",
  "title": "Connecting to the node"
}
[/block]
First you need to connect to configured Ignite node using ODBC. Nothing new here - you should properly specify connection string arguments. If some argument was not specified default value is used instead. Refer to [Connection String](doc:connecting-string) page for details. Pay special attention to `Cache` attribute - you should specify name of any existing cache here. It is not really important which one thanks to [Cross-cache queries](doc:sql-queries#cross-cache-queries). Note however that you have to specify schema name prior to your tables if you are trying to run query upon some table of a cache which is not the one you connected to.

You can also use pre-configured DSN for connection.
[block:code]
{
  "codes": [
    {
      "code": "SQLHENV env;\n\n// Allocate an environment handle\nSQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &env);\n\n// We want ODBC 3 support\nSQLSetEnvAttr(env, SQL_ATTR_ODBC_VERSION, reinterpret_cast<void*>(SQL_OV_ODBC3), 0);\n\nSQLHDBC dbc;\n\n// Allocate a connection handle\nSQLAllocHandle(SQL_HANDLE_DBC, env, &dbc);\n\n// Connection string\nSQLCHAR connectStr[] = \"DSN=My Ignite DSN\";\n\n// Connecting to ODBC server.\nSQLRETURN ret = SQLDriverConnect(dbc, NULL, connectStr, SQL_NTS, NULL, 0, NULL, SQL_DRIVER_COMPLETE);\n\nif (!SQL_SUCCEEDED(ret))\n{\n  SQLCHAR sqlstate[7] = { 0 };\n  SQLINTEGER nativeCode;\n\n  SQLCHAR errMsg[BUFFER_SIZE] = { 0 };\n  SQLSMALLINT errMsgLen = static_cast<SQLSMALLINT>(sizeof(errMsg));\n\n  SQLGetDiagRec(SQL_HANDLE_DBC, dbc, 1, sqlstate, &nativeCode, errMsg, errMsgLen, &errMsgLen);\n  \n  std::cerr << \"Failed to connect to Apache Ignite: \" \n            << reinterpret_cast<char*>(sqlstate) << \": \"\n            << reinterpret_cast<char*>(errMsg) << \", \"\n            << \"Native error code: \" << nativeCode \n            << std::endl;\n\n  // Releasing allocated handles.\n  SQLFreeHandle(SQL_HANDLE_DBC, dbc);\n  SQLFreeHandle(SQL_HANDLE_ENV, env);\n  \n  return;\n}",
      "language": "cplusplus"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Inserting records"
}
[/block]
Lets insert some records using `INSERT` query into `Person` cache. Here we are going to prepare statement and use parameters.
[block:callout]
{
  "type": "info",
  "title": "There is no error checking in the code below",
  "body": "This is to make code more clear and easier to understand. You should always check for errors in production."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "SQLHSTMT stmt;\n\n// Allocate a statement handle\nSQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\nSQLCHAR query[] =\n\t\"INSERT INTO Person (_key, orgId, firstName, lastName, resume, salary) \"\n\t\"VALUES (?, ?, ?, ?, ?, ?)\";\n\nSQLPrepare(stmt, query, static_cast<SQLSMALLINT>(sizeof(query)));\n\n// Binding columns.\nint64_t key = 0;\nint64_t orgId = 0;\nchar name[1024] = { 0 };\nSQLLEN nameLen = SQL_NTS;\ndouble salary = 0.0;\n\nSQLBindParameter(stmt, 1, SQL_PARAM_INPUT, SQL_C_SLONG, SQL_BIGINT, 0, 0, &key, 0, 0);\nSQLBindParameter(stmt, 2, SQL_PARAM_INPUT, SQL_C_SLONG, SQL_BIGINT, 0, 0, &orgId, 0, 0);\nSQLBindParameter(stmt, 3, SQL_PARAM_INPUT, SQL_C_CHAR, SQL_VARCHAR,\tsizeof(name), sizeof(name), name, 0, &nameLen);\nSQLBindParameter(stmt, 4, SQL_PARAM_INPUT, SQL_C_DOUBLE, SQL_DOUBLE, 0, 0, &salary, 0, 0);\n\n// Filling cache.\nkey = 1;\norgId = 1;\nstrncpy(name, \"John\", sizeof(firstName));\nsalary = 2200.0;\n\nSQLExecute(stmt);\nSQLMoreResults(stmt);\n\n++key;\norgId = 1;\nstrncpy(name, \"Jane\", sizeof(firstName));\nsalary = 1300.0;\n\nSQLExecute(stmt);\nSQLMoreResults(stmt);\n\n++key;\norgId = 2;\nstrncpy(name, \"Richard\", sizeof(firstName));\nsalary = 900.0;\n\nSQLExecute(stmt);\nSQLMoreResults(stmt);\n\n++key;\norgId = 2;\nstrncpy(name, \"Mary\", sizeof(firstName));\nsalary = 2400.0;\n\nSQLExecute(stmt);\nSQLMoreResults(stmt);",
      "language": "cplusplus"
    }
  ]
}
[/block]
Next we are going to insert a few organizations without preparing statements.
[block:code]
{
  "codes": [
    {
      "code": "SQLHSTMT stmt;\n\n// Allocate a statement handle\nSQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\nSQLCHAR query1[] = \"INSERT INTO \\\"Organization\\\".Organization (_key, name) VALUES (1L, 'Some company')\";\nSQLExecDirect(stmt, query1, static_cast<SQLSMALLINT>(sizeof(query1)));\nSQLFreeStmt(stmt, SQL_CLOSE);\n\nSQLCHAR query2[] = \"INSERT INTO \\\"Organization\\\".Organization (_key, name) VALUES (2L, 'Some other company')\";\nSQLExecDirect(stmt, query2, static_cast<SQLSMALLINT>(sizeof(query2)));\nSQLFreeStmt(stmt, SQL_CLOSE);",
      "language": "cplusplus"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Updating records"
}
[/block]
Now lets adjust someones salary. Lets write simple function for that.
[block:code]
{
  "codes": [
    {
      "code": "void AdjustSalary(SQLHDBC dbc, int64_t key, double salary)\n{\n  SQLHSTMT stmt;\n\n  // Allocate a statement handle\n  SQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\n  SQLCHAR query[] = \"UPDATE Person SET salary=? WHERE _key=?\";\n\n  SQLBindParameter(stmt, 1, SQL_PARAM_INPUT, SQL_C_DOUBLE, SQL_DOUBLE, 0, 0, &salary, 0, 0);\n  SQLBindParameter(stmt, 2, SQL_PARAM_INPUT, SQL_C_SLONG, SQL_BIGINT, 0, 0, &key, 0, 0);\n\n  SQLExecDirect(stmt, query, static_cast<SQLSMALLINT>(sizeof(query)));\n\n  // Releasing statement handle.\n  SQLFreeHandle(SQL_HANDLE_STMT, stmt);\n}\n\n...\nAdjustSalary(dbc, 3, 1200.0);\nAdjustSalary(dbc, 1, 2500.0);",
      "language": "cplusplus"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Deleting records"
}
[/block]
Finally, lets remove a few records. Lets use a function for this again:
[block:code]
{
  "codes": [
    {
      "code": "void DeletePerson(SQLHDBC dbc, int64_t key)\n{\n  SQLHSTMT stmt;\n\n  // Allocate a statement handle\n  SQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\n  SQLCHAR query[] = \"DELETE FROM Person WHERE _key=?\";\n\n  SQLBindParameter(stmt, 1, SQL_PARAM_INPUT, SQL_C_SLONG, SQL_BIGINT, 0, 0, &key, 0, 0);\n\n  SQLExecDirect(stmt, query, static_cast<SQLSMALLINT>(sizeof(query)));\n\n  // Releasing statement handle.\n  SQLFreeHandle(SQL_HANDLE_STMT, stmt);\n}\n\n...\nDeletePerson(dbc, 1);\nDeletePerson(dbc, 4);",
      "language": "cplusplus"
    }
  ]
}
[/block]