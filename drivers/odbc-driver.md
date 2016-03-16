Ignite now includes ODBC driver that allows you to retrieve distributed data from cache using standard SQL queries and native ODBC API.

For detailed info on ODBC please refer to [ODBC Programmer's Reference](https://msdn.microsoft.com/en-us/library/ms714177.aspx)
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
    "2-0": "Visual Studio",
    "2-1": "2010 and above"
  },
  "cols": 2,
  "rows": 3
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Connection string and DSN arguments"
}
[/block]
Apache Ignite ODBC driver supports and uses following connection string/DSN
arguments:
[block:parameters]
{
  "data": {
    "h-0": "Parameter",
    "h-1": "Description",
    "h-2": "Default value",
    "0-0": "SERVER",
    "1-0": "PORT",
    "2-0": "CACHE",
    "2-1": "Cache name. If it is not defined than default cache will be used. Note that the cache name is case sensitive.",
    "1-1": "Port on which OdbcProcessor of the node is listening.",
    "0-1": "Address of the node to connect to.",
    "0-2": "lacalhost",
    "1-2": "11443"
  },
  "cols": 3,
  "rows": 3
}
[/block]
All parameter names are case-insensitive so `SERVER`, `Server` and `server` all are
valid parameter names and would set the same parameter.
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
  "title": "Joins and Collocation",
  "body": "Just like with [Cache SQL Queries](doc:cache-queries) used from `IgniteCache` API, joins on `PARTITIONED` caches will work correctly only if joined objects are stored in collocated mode. Refer to [Affinity Collocation](/docs/affinity-collocation#collocate-data-with-data) for more details."
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
  "title": "ODBC Interface Conformance"
}
[/block]
ODBC [defines](https://msdn.microsoft.com/en-us/library/ms710289.aspx) several Interface conformance levels. In this section you can find which features are supported by the Ignite Apache ODBC driver.

##Core Interface conformance
[block:parameters]
{
  "data": {
    "h-0": "Feature",
    "h-1": "Supported in Apache Ignite",
    "h-2": "Comments",
    "0-0": "Allocate and free all types of handles, by calling SQLAllocHandle and SQLFreeHandle.",
    "0-1": "Yes",
    "1-0": "Use all forms of the SQLFreeStmt function.",
    "1-1": "Yes",
    "2-0": "Bind result set columns, by calling SQLBindCol.",
    "2-1": "Yes",
    "4-0": "Specify a bind offset.",
    "4-1": "Yes",
    "5-0": "Use the data-at-execution dialog, involving calls to SQLParamData and SQLPutData",
    "5-1": "No",
    "5-2": "Mainly used for INSERT and UPDATE queries, which are not supported by Apache Ignite.",
    "6-0": "Manage cursors and cursor names, by calling SQLCloseCursor, SQLGetCursorName, and SQLSetCursorName.",
    "6-1": "Partially",
    "6-2": "SQLCloseCursor is implemented. Named cursors are not supported by Apache Ignite.",
    "7-0": "Gain access to the description (metadata) of result sets, by calling SQLColAttribute, SQLDescribeCol, SQLNumResultCols, and SQLRowCount.",
    "7-1": "Yes",
    "8-0": "Query the data dictionary, by calling the catalog functions SQLColumns, SQLGetTypeInfo, SQLStatistics, and SQLTables.",
    "8-1": "Partially",
    "8-2": "SQLStatistics is not supported.",
    "9-0": "Manage data sources and connections, by calling SQLConnect, SQLDataSources, SQLDisconnect, and SQLDriverConnect. Obtain information on drivers, no matter which ODBC level they support, by calling SQLDrivers.",
    "9-1": "Yes",
    "12-0": "Obtain an unbound column in parts, by calling SQLGetData.",
    "12-1": "Yes",
    "13-0": "Obtain current values of all attributes, by calling SQLGetConnectAttr, SQLGetEnvAttr, and SQLGetStmtAttr, and set all attributes to their default values and set certain attributes to nondefault values by calling SQLSetConnectAttr, SQLSetEnvAttr, and SQLSetStmtAttr.",
    "13-1": "Partially",
    "13-2": "Not all the attributes are supported.",
    "14-0": "Manipulate certain fields of descriptors, by calling SQLCopyDesc, SQLGetDescField, SQLGetDescRec, SQLSetDescField, and SQLSetDescRec.",
    "14-1": "No",
    "15-0": "Obtain diagnostic information, by calling SQLGetDiagField and SQLGetDiagRec.",
    "15-1": "Yes",
    "16-0": "Detect driver capabilities, by calling SQLGetFunctions and SQLGetInfo. Also, detect the result of any text substitutions made to an SQL statement before it is sent to the data source, by calling SQLNativeSql.",
    "16-1": "Partially",
    "16-2": "SQLGetFunctions is not implemented, SQLGetInfo implemented partially, SQLNativeSql is implemented.",
    "17-0": "Use the syntax of SQLEndTran to commit a transaction. A Core-level driver need not support true transactions; therefore, the application cannot specify SQL_ROLLBACK nor SQL_AUTOCOMMIT_OFF for the SQL_ATTR_AUTOCOMMIT connection attribute.",
    "17-1": "Yes",
    "18-0": "Call SQLCancel to cancel the data-at-execution dialog and, in multithread environments, to cancel an ODBC function executing in another thread. Core-level interface conformance does not mandate support for asynchronous execution of functions, nor the use of SQLCancel to cancel an ODBC function executing asynchronously. Neither the platform nor the ODBC driver need be multithread for the driver to conduct independent activities at the same time. However, in multithread environments, the ODBC driver must be thread-safe. Serialization of requests from the application is a conformant way to implement this specification, even though it might create serious performance problems.",
    "18-1": "No",
    "18-2": "Current implementation of the ODBC driver does not support asynchronous execution nor data-at-execution feature.",
    "19-0": "Obtain the SQL_BEST_ROWID row-identifying column of tables, by calling SQLSpecialColumns.",
    "19-1": "Yes",
    "19-2": "Current implementation always returns empty row set.",
    "3-0": "Handle dynamic parameters, including arrays of parameters, in the input direction only, by calling SQLBindParameter and SQLNumParams.",
    "3-1": "Yes",
    "10-0": "Prepare and execute SQL statements, by calling SQLExecDirect, SQLExecute, and SQLPrepare.",
    "10-1": "Yes",
    "11-0": "Fetch one row of a result set or multiple rows, in the forward direction only, by calling SQLFetch or by calling SQLFetchScroll with the FetchOrientation argument set to SQL_FETCH_NEXT",
    "11-1": "Yes"
  },
  "cols": 3,
  "rows": 20
}
[/block]
##Function support
[block:parameters]
{
  "data": {
    "h-0": "Function",
    "h-1": "Supported in Apache Ignite",
    "h-2": "Conformance level",
    "0-0": "SQLAllocHandle",
    "0-1": "YES",
    "1-1": "YES",
    "1-0": "SQLBindCol",
    "59-0": "SQLTables",
    "58-0": "SQLTablePrivileges",
    "57-0": "SQLStatistics",
    "2-0": "SQLBindParameter",
    "3-0": "SQLBrowseConnect",
    "4-0": "SQLBulkOperations",
    "5-0": "SQLCancel",
    "6-0": "SQLCloseCursor",
    "7-0": "SQLColAttribute",
    "8-0": "SQLColumnPrivileges",
    "9-0": "SQLColumns",
    "10-0": "SQLConnect",
    "11-0": "SQLCopyDesc",
    "12-0": "SQLDataSources",
    "13-0": "SQLDescribeCol",
    "14-0": "SQLDescribeParam",
    "15-0": "SQLDisconnect",
    "16-0": "SQLDriverConnect",
    "17-0": "SQLDrivers",
    "18-0": "SQLEndTran",
    "19-0": "SQLExecDirect",
    "20-0": "SQLExecute",
    "21-0": "SQLFetch",
    "22-0": "SQLFetchScroll",
    "23-0": "SQLForeignKeys",
    "24-0": "SQLFreeHandle",
    "25-0": "SQLFreeStmt",
    "26-0": "SQLGetConnectAttr",
    "27-0": "SQLGetCursorName",
    "28-0": "SQLGetData",
    "29-0": "SQLGetDescField",
    "30-0": "SQLGetDescRec",
    "31-0": "SQLGetDiagField",
    "32-0": "SQLGetDiagRec",
    "33-0": "SQLGetEnvAttr",
    "34-0": "SQLGetFunctions",
    "35-0": "SQLGetInfo",
    "36-0": "SQLGetStmtAttr",
    "37-0": "SQLGetTypeInfo",
    "38-0": "SQLMoreResults",
    "39-0": "SQLNativeSql",
    "40-0": "SQLNumParams",
    "41-0": "SQLNumResultCols",
    "42-0": "SQLParamData",
    "43-0": "SQLPrepare",
    "44-0": "SQLPrimaryKeys",
    "45-0": "SQLProcedureColumns",
    "46-0": "SQLProcedures",
    "47-0": "SQLPutData",
    "48-0": "SQLRowCount",
    "49-0": "SQLSetConnectAttr",
    "50-0": "SQLSetCursorName",
    "51-0": "SQLSetDescField",
    "52-0": "SQLSetDescRec",
    "53-0": "SQLSetEnvAttr",
    "54-0": "SQLSetPos",
    "55-0": "SQLSetStmtAttr",
    "56-0": "SQLSpecialColumns",
    "2-1": "YES",
    "59-1": "YES",
    "3-1": "NO",
    "4-1": "NO",
    "5-1": "NO",
    "6-1": "YES",
    "7-1": "YES",
    "8-1": "NO",
    "9-1": "YES",
    "10-1": "YES",
    "11-1": "NO",
    "12-1": "N/A",
    "13-1": "YES",
    "14-1": "NO",
    "15-1": "YES",
    "16-1": "YES",
    "17-1": "N/A",
    "18-1": "PARTIALLY",
    "19-1": "YES",
    "20-1": "YES",
    "21-1": "YES",
    "22-1": "YES",
    "23-1": "PARTIALLY",
    "24-1": "YES",
    "25-1": "YES",
    "26-1": "NO",
    "27-1": "NO",
    "58-1": "NO",
    "28-1": "YES",
    "29-1": "NO",
    "30-1": "NO",
    "31-1": "YES",
    "32-1": "YES",
    "33-1": "YES",
    "34-1": "NO",
    "35-1": "YES",
    "36-1": "PARTIALLY",
    "37-1": "YES",
    "38-1": "PARTIALLY",
    "39-1": "YES",
    "40-1": "YES",
    "41-1": "YES",
    "42-1": "NO",
    "43-1": "YES",
    "44-1": "YES",
    "45-1": "NO",
    "46-1": "NO",
    "47-1": "NO",
    "48-1": "YES",
    "49-1": "NO",
    "50-1": "NO",
    "51-1": "NO",
    "52-1": "NO",
    "53-1": "YES",
    "54-1": "NO",
    "55-1": "PARTIALLY",
    "56-1": "NO",
    "57-1": "NO",
    "0-2": "Core",
    "59-2": "Core",
    "23-2": "Level 2",
    "1-2": "Core",
    "2-2": "Core",
    "3-2": "Level 1",
    "4-2": "Level 1",
    "5-2": "Core",
    "6-2": "Core",
    "7-2": "Core",
    "8-2": "Level 2",
    "9-2": "Core",
    "10-2": "Core",
    "11-2": "Core",
    "12-2": "Core",
    "13-2": "Core",
    "14-2": "Level 2",
    "15-2": "Core",
    "16-2": "Core",
    "17-2": "Core",
    "18-2": "Core",
    "19-2": "Core",
    "20-2": "Core",
    "21-2": "Core",
    "22-2": "Core",
    "24-2": "Core",
    "25-2": "Core",
    "26-2": "Core",
    "27-2": "Core",
    "28-2": "Core",
    "29-2": "Core",
    "30-2": "Core",
    "31-2": "Core",
    "32-2": "Core",
    "33-2": "Core",
    "34-2": "Core",
    "35-2": "Core",
    "36-2": "Core",
    "37-2": "Core",
    "38-2": "Level 1",
    "39-2": "Core",
    "40-2": "Core",
    "41-2": "Core",
    "42-2": "Core",
    "43-2": "Core",
    "44-2": "Level 1",
    "45-2": "Level 1",
    "46-2": "Level 1",
    "47-2": "Core",
    "48-2": "Core",
    "49-2": "Core",
    "50-2": "Core",
    "51-2": "Core",
    "52-2": "Core",
    "53-2": "Core",
    "54-2": "Level 1",
    "55-2": "Core",
    "56-2": "Core",
    "57-2": "Core",
    "58-2": "Level 2",
    "h-3": "Comments",
    "12-3": "Driver Manager function.",
    "17-3": "Driver Manager function.",
    "23-3": "Implemented but returns empty data.",
    "36-3": "Not all attributes are supported. See Statement Attribute Conformance below.",
    "55-3": "",
    "38-3": "Implemented for basic case.",
    "39-3": "Currently returns unchanged input data."
  },
  "cols": 3,
  "rows": 60
}
[/block]