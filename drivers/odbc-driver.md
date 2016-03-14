Ignite now includes ODBC driver that allows you to retrieve distributed data from cache using standard SQL queries and native ODBC API.

For more info on ODBC please refer to [ODBC Programmer's Reference](https://msdn.microsoft.com/en-us/library/ms714177.aspx)
[block:api-header]
{
  "type": "basic",
  "title": "ODBC Interface Conformance"
}
[/block]
ODBC [defines](https://msdn.microsoft.com/en-us/library/ms710289.aspx) several Interface conformance levels. In this section you can find which features are supported by the Ignite Apache ODBC driver.

###Core Interface conformance
[block:parameters]
{
  "data": {
    "h-0": "Feature",
    "h-1": "Supported in Apache Ignite",
    "h-2": "Comments",
    "0-0": "Allocate and free all types of handles, by calling SQLAllocHandle and SQLFreeHandle.",
    "0-1": "Yes"
  },
  "cols": 3,
  "rows": 2
}
[/block]