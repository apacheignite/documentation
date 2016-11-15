* [Configuring Ignite Cluster](#configuring-ignite-cluster)
* [Connecting to the Cluster](#connecting-to-the-cluster)
* [Selecting records](#selecting-records)
* [Inserting records](#inserting-records)
* [Updating records](#updating-records)
* [Deleting records](#deleting-records)

This page elaborates on how to connect to an Ignite cluster and execute a variety of SQL queries using Ignite's ODBC driver. 

At the implementation layer Ignite's ODBC driver uses Ignite SQL Fields queries to retrieve data from  Apache Ignite Data Grid. This means that from ODBC side you can access only those fields that are [defined in cluster's configuration](/docs/sql-queries#section-making-fields-visible-for-sql-queries).

Moreover, starting from Ignite 1.8 the ODBC driver supports DML (Data Modification Layer) which means that you can not only request data but modify Ignite Data Grid content as well using an ODBC connection.
[block:callout]
{
  "type": "success",
  "body": "Refer to [ODBC Example](https://github.com/apache/ignite/tree/master/modules/platforms/cpp/examples/odbc-example) that incorporates complete logic and exemplary queries shown below in the documentation."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring Ignite Cluster"
}
[/block]
As the first step, you need to set up a configuration that will be used by the cluster nodes. The configuration should include caches configurations as well with properly defined `QueryEntities` properties. `QueryEntities` are essential for the cases when an application (or the ODBC driver in our scenario) is going to query and modify data using SQL statements.
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\n        http://www.springframework.org/schema/util\n        http://www.springframework.org/schema/util/spring-util.xsd\">\n  <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <!-- Enabling ODBC. -->\n    <property name=\"odbcConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.OdbcConfiguration\"/>\n    </property>\n\n    <!-- Configuring cache. -->\n    <property name=\"cacheConfiguration\">\n      <list>\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Person\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Person\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"name\" value=\"java.lang.String\"/>\n                    <entry key=\"salary\" value=\"java.lang.Double\"/>\n                  </map>\n                </property>\n                \n                <property name=\"indexes\">\n                    <list>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"salary\"/>\n                        </bean>\n                    </list>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n        \n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Organization\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Organization\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"name\" value=\"java.lang.String\"/>\n                  </map>\n                </property>\n                \n                <property name=\"indexes\">\n                    <list>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"name\"/>\n                        </bean>\n                    </list>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n      </list>\n    </property>\n  </bean>\n</beans>\n",
      "language": "xml",
      "name": "XML"
    }
  ]
}
[/block]
As you can see from the configuration, we defined two Ignite caches that will contain data of  `Person` and `Organization` types. For both of the types we listed specific fields and indexes that will be read or updated using SQL.
[block:callout]
{
  "type": "info",
  "body": "In addition to all the explicitly configured fields, each table will have two special predefined fields: `_key` and `_val`, which represent links to whole key and value objects. This is useful, for example, when one of them is a primitive value and you want to filter out records by its value. To do this, execute a query like `SELECT * FROM Person WHERE _key = 100`.",
  "title": "Predefined Fields"
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "OdbcConfiguration",
  "body": "Make sure that `OdbcConfiguration` is explicitly set in the configuration. You can learn more about its importance by referring to [this](/docs/getting-started-18#cluster-configuration) documentation section."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Connecting to the Cluster"
}
[/block]
After the cluster is configured and started we can connect to it from the ODBC driver side. To do this you need to prepare a valid connection string and pass it as a parameter to the ODBC driver at the connection time. Refer to [Connection String](doc:connecting-string) page for more details.

Alternatively, you can also use [pre-configured DSN](connection-string-and-dsn#configuring-dsn) for connection purposes as it's shown in the example below.
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
  "title": "Selecting records"
}
[/block]
Now, when everything set up and ready we can use ODBC API to run SQL query upon our data grid just like if it was ordinary database. Lets try it out.
[block:code]
{
  "codes": [
    {
      "code": "SQLHSTMT stmt;\n\n// Allocate a statement handle\nSQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\nSQLCHAR query[] = \"SELECT firstName, lastName, salary, Organization.name FROM Person \"\n  \"INNER JOIN \\\"Organization\\\".Organization ON Person.orgId = Organization._key\";\nSQLSMALLINT queryLen = static_cast<SQLSMALLINT>(sizeof(queryLen));\n\nSQLRETURN ret = SQLExecDirect(stmt, query, queryLen);\n\nif (!SQL_SUCCEEDED(ret))\n{\n  SQLCHAR sqlstate[7] = { 0 };\n  SQLINTEGER nativeCode;\n\n  SQLCHAR errMsg[BUFFER_SIZE] = { 0 };\n  SQLSMALLINT errMsgLen = static_cast<SQLSMALLINT>(sizeof(errMsg));\n\n  SQLGetDiagRec(SQL_HANDLE_DBC, dbc, 1, sqlstate, &nativeCode, errMsg, errMsgLen, &errMsgLen);\n  \n  std::cerr << \"Failed to perfrom SQL query upon Apache Ignite: \" \n            << reinterpret_cast<char*>(sqlstate) << \": \"\n            << reinterpret_cast<char*>(errMsg) << \", \"\n            << \"Native error code: \" << nativeCode \n            << std::endl;\n}\nelse\n{\n  // Printing results.\n  \n  struct OdbcStringBuffer\n  {\n    SQLCHAR buffer[BUFFER_SIZE];\n    SQLLEN resLen;\n  };\n  \n  // Getting number of columns in result set.\n  SQLSMALLINT columnsCnt = 0;\n  SQLNumResultCols(stmt, &columnsCnt);\n\n  // Allocating buffers for columns.\n  std::vector<OdbcStringBuffer> columns(columnsCnt);\n\n  // Binding colums. For simplicity we are going to use only\n  // string buffers here.\n  for (SQLSMALLINT i = 0; i < columnsCnt; ++i)\n    SQLBindCol(stmt, i + 1, SQL_C_CHAR, columns[i].buffer, BUFFER_SIZE, &columns[i].resLen);\n\n  // Fetching and printing data in a loop.\n  ret = SQLFetch(stmt);\n  while (SQL_SUCCEEDED(ret))\n  {\n    for (size_t i = 0; i < columns.size(); ++i)\n      std::cout << std::setw(16) << std::left << columns[i].buffer << \" \";\n\n    std::cout << std::endl;\n    \n    ret = SQLFetch(stmt);\n  }\n}\n\n// Releasing statement handle.\nSQLFreeHandle(SQL_HANDLE_STMT, stmt);",
      "language": "cplusplus"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "In the example above we bind all columns to the `SQL_C_CHAR` columns. This means all values are going to be converted to strings upon fetching. We only do that for the sake of the simplicity. Converting values upon fetching can be pretty slow so your default decision should be to fetch value the same way its stored.",
  "title": "Columns binding"
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
      "code": "SQLHSTMT stmt;\n\n// Allocate a statement handle\nSQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\nSQLCHAR query[] =\n\t\"INSERT INTO Person (_key, orgId, firstName, lastName, resume, salary) \"\n\t\"VALUES (?, ?, ?, ?, ?, ?)\";\n\nSQLPrepare(stmt, query, static_cast<SQLSMALLINT>(sizeof(query)));\n\n// Binding columns.\nint64_t key = 0;\nint64_t orgId = 0;\nchar name[1024] = { 0 };\nSQLLEN nameLen = SQL_NTS;\ndouble salary = 0.0;\n\nSQLBindParameter(stmt, 1, SQL_PARAM_INPUT, SQL_C_SLONG, SQL_BIGINT, 0, 0, &key, 0, 0);\nSQLBindParameter(stmt, 2, SQL_PARAM_INPUT, SQL_C_SLONG, SQL_BIGINT, 0, 0, &orgId, 0, 0);\nSQLBindParameter(stmt, 3, SQL_PARAM_INPUT, SQL_C_CHAR, SQL_VARCHAR,\tsizeof(name), sizeof(name), name, 0, &nameLen);\nSQLBindParameter(stmt, 4, SQL_PARAM_INPUT, SQL_C_DOUBLE, SQL_DOUBLE, 0, 0, &salary, 0, 0);\n\n// Filling cache.\nkey = 1;\norgId = 1;\nstrncpy(name, \"John\", sizeof(firstName));\nsalary = 2200.0;\n\nSQLExecute(stmt);\nSQLMoreResults(stmt);\n\n++key;\norgId = 1;\nstrncpy(name, \"Jane\", sizeof(firstName));\nsalary = 1300.0;\n\nSQLExecute(stmt);\nSQLMoreResults(stmt);\n\n++key;\norgId = 2;\nstrncpy(name, \"Richard\", sizeof(firstName));\nsalary = 900.0;\n\nSQLExecute(stmt);\nSQLMoreResults(stmt);\n\n++key;\norgId = 2;\nstrncpy(name, \"Mary\", sizeof(firstName));\nsalary = 2400.0;\n\nSQLExecute(stmt);\n\n// Releasing statement handle.\nSQLFreeHandle(SQL_HANDLE_STMT, stmt);",
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
      "code": "SQLHSTMT stmt;\n\n// Allocate a statement handle\nSQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\nSQLCHAR query1[] = \"INSERT INTO \\\"Organization\\\".Organization (_key, name) VALUES (1L, 'Some company')\";\nSQLExecDirect(stmt, query1, static_cast<SQLSMALLINT>(sizeof(query1)));\nSQLFreeStmt(stmt, SQL_CLOSE);\n\nSQLCHAR query2[] = \"INSERT INTO \\\"Organization\\\".Organization (_key, name) VALUES (2L, 'Some other company')\";\nSQLExecDirect(stmt, query2, static_cast<SQLSMALLINT>(sizeof(query2)));\n\n// Releasing statement handle.\nSQLFreeHandle(SQL_HANDLE_STMT, stmt);",
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