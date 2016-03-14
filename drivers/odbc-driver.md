Ignite now includes ODBC driver that allows you to retrieve distributed data from cache using standard SQL queries and native ODBC API.

For more info on ODBC please refer to [ODBC Programmer's Reference](https://msdn.microsoft.com/en-us/library/ms714177.aspx)
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
    "5-2": "Cannot be impemented: Ignite does not currently support statement preparing. But as it mainly used for INSERT and UPDATE queries, there is not much sense in implementing anyway.",
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
    "10-0": "Obtain an unbound column in parts, by calling SQLGetData.",
    "10-1": "Yes",
    "11-0": "Obtain current values of all attributes, by calling SQLGetConnectAttr, SQLGetEnvAttr, and SQLGetStmtAttr, and set all attributes to their default values and set certain attributes to nondefault values by calling SQLSetConnectAttr, SQLSetEnvAttr, and SQLSetStmtAttr.",
    "11-1": "Partially",
    "11-2": "Not all the attributes are supported.",
    "12-0": "Manipulate certain fields of descriptors, by calling SQLCopyDesc, SQLGetDescField, SQLGetDescRec, SQLSetDescField, and SQLSetDescRec.",
    "12-1": "No",
    "13-0": "Obtain diagnostic information, by calling SQLGetDiagField and SQLGetDiagRec.",
    "13-1": "Yes",
    "14-0": "Detect driver capabilities, by calling SQLGetFunctions and SQLGetInfo. Also, detect the result of any text substitutions made to an SQL statement before it is sent to the data source, by calling SQLNativeSql.",
    "14-1": "Partially",
    "14-2": "SQLGetFunctions is not implemented, SQLGetInfo implemented partially, SQLNativeSql is implemented.",
    "15-0": "Use the syntax of SQLEndTran to commit a transaction. A Core-level driver need not support true transactions; therefore, the application cannot specify SQL_ROLLBACK nor SQL_AUTOCOMMIT_OFF for the SQL_ATTR_AUTOCOMMIT connection attribute.",
    "15-1": "Yes",
    "16-0": "Call SQLCancel to cancel the data-at-execution dialog and, in multithread environments, to cancel an ODBC function executing in another thread. Core-level interface conformance does not mandate support for asynchronous execution of functions, nor the use of SQLCancel to cancel an ODBC function executing asynchronously. Neither the platform nor the ODBC driver need be multithread for the driver to conduct independent activities at the same time. However, in multithread environments, the ODBC driver must be thread-safe. Serialization of requests from the application is a conformant way to implement this specification, even though it might create serious performance problems.",
    "16-1": "No",
    "16-2": "Current implementation of the ODBC driver does not support asynchronous execution nor data-at-execution feature.",
    "17-0": "Obtain the SQL_BEST_ROWID row-identifying column of tables, by calling SQLSpecialColumns.",
    "17-1": "Yes",
    "17-2": "Current implementation always returns empty row set.",
    "3-0": "Handle dynamic parameters, including arrays of parameters, in the input direction only, by calling SQLBindParameter and SQLNumParams.",
    "3-1": "Yes"
  },
  "cols": 3,
  "rows": 18
}
[/block]