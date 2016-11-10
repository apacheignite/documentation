Starting from Ignite 1.8, DML is supported in SQL queries. This means DML can now be used in ODBC as well. On this you can find details.
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
      "code": "#define BUFFER_SIZE 1024\n\nSQLHENV env;\n\n// Allocate an environment handle\nSQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &env);\n\n// We want ODBC 3 support\nSQLSetEnvAttr(env, SQL_ATTR_ODBC_VERSION, reinterpret_cast<void*>(SQL_OV_ODBC3), 0);\n\nSQLHDBC dbc;\n\n// Allocate a connection handle\nSQLAllocHandle(SQL_HANDLE_DBC, env, &dbc);\n\n// Connection string\nSQLCHAR connectStr[] = \"DSN=My Ignite DSN\";\n\nSQLCHAR outStr[BUFFER_SIZE] = { 0 };\nSQLSMALLINT outStrLen = static_cast<SQLSMALLINT>(sizeof(outStr));;\n\n// Connecting to ODBC server.\nSQLRETURN ret = SQLDriverConnect(dbc, NULL, connectStr, SQL_NTS, outStr, outStrLen, &outStrLen, SQL_DRIVER_COMPLETE);\n\nif (!SQL_SUCCEEDED(ret))\n{\n  SQLCHAR sqlstate[7] = { 0 };\n  SQLINTEGER nativeCode;\n\n  SQLCHAR errMsg[BUFFER_SIZE] = { 0 };\n  SQLSMALLINT errMsgLen = static_cast<SQLSMALLINT>(sizeof(errMsg));\n\n  SQLGetDiagRec(SQL_HANDLE_DBC, dbc, 1, sqlstate, &nativeCode, errMsg, errMsgLen, &errMsgLen);\n  \n  std::cerr << \"Failed to connect to Apache Ignite: \" \n            << reinterpret_cast<char*>(sqlstate) << \": \"\n            << reinterpret_cast<char*>(errMsg) << \", \"\n            << \"Native error code: \" << nativeCode \n            << std::endl;\n\n  // Releasing allocated handles.\n  SQLFreeHandle(SQL_HANDLE_DBC, dbc);\n  SQLFreeHandle(SQL_HANDLE_ENV, env);\n  \n  return;\n}\n\nSQLHSTMT stmt;\n\n// Allocate a statement handle\nSQLAllocHandle(SQL_HANDLE_STMT, dbc, &stmt);\n\nSQLCHAR query[] = \"SELECT name, salary, Organization.name FROM Person \"\n  \"INNER JOIN \\\"Organization\\\".Organization ON Person.orgId = Organization.id\";\nSQLSMALLINT queryLen = static_cast<SQLSMALLINT>(sizeof(queryLen));\n\nret = SQLExecDirect(stmt, query, queryLen);\n\nif (!SQL_SUCCEEDED(ret))\n{\n  SQLCHAR sqlstate[7] = { 0 };\n  SQLINTEGER nativeCode;\n\n  SQLCHAR errMsg[BUFFER_SIZE] = { 0 };\n  SQLSMALLINT errMsgLen = static_cast<SQLSMALLINT>(sizeof(errMsg));\n\n  SQLGetDiagRec(SQL_HANDLE_DBC, dbc, 1, sqlstate, &nativeCode, errMsg, errMsgLen, &errMsgLen);\n  \n  std::cerr << \"Failed to perfrom SQL query upon Apache Ignite: \" \n            << reinterpret_cast<char*>(sqlstate) << \": \"\n            << reinterpret_cast<char*>(errMsg) << \", \"\n            << \"Native error code: \" << nativeCode \n            << std::endl;\n}\nelse\n{\n  // Printing results.\n  \n  struct OdbcStringBuffer\n  {\n    SQLCHAR buffer[BUFFER_SIZE];\n    SQLLEN resLen;\n  };\n  \n  // Getting number of columns in result set.\n  SQLSMALLINT columnsCnt = 0;\n  SQLNumResultCols(stmt, &columnsCnt);\n\n  // Allocating buffers for columns.\n  std::vector<OdbcStringBuffer> columns(columnsCnt);\n\n  // Binding colums. For simplicity we are going to use only\n  // string buffers here.\n  for (SQLSMALLINT i = 0; i < columnsCnt; ++i)\n    SQLBindCol(stmt, i + 1, SQL_CHAR, columns[i].buffer, BUFFER_SIZE, &columns[i].resLen);\n\n  // Fetching and printing data in a loop.\n  ret = SQLFetch(stmt);\n  while (SQL_SUCCEEDED(ret))\n  {\n    for (size_t i = 0; i < columns.size(); ++i)\n      std::cout << std::setw(16) << std::left << columns[i].buffer << \" \";\n\n    std::cout << std::endl;\n    \n    ret = SQLFetch(stmt);\n  }\n}\n\n// Releasing statement handle.\nSQLFreeHandle(SQL_HANDLE_STMT, stmt);\n\n// Disconneting from the server.\nSQLDisconnect(dbc);\n\n// Releasing allocated handles.\nSQLFreeHandle(SQL_HANDLE_DBC, dbc);\nSQLFreeHandle(SQL_HANDLE_ENV, env);",
      "language": "cplusplus"
    }
  ]
}
[/block]