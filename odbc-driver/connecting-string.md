* [Connection String Format](#connection-string-format)
* [Supported Arguments](#supported-arguments)
* [Connection String Example](#connection-string-example)
[block:api-header]
{
  "type": "basic",
  "title": "Connection string format"
}
[/block]
Apache Ignite ODBC Driver supports standard connection string format. Here is the formal syntax:

```
connection-string ::= empty-string[;] | attribute[;] | attribute; connection-string
empty-string ::=
attribute ::= attribute-keyword=attribute-value | DRIVER=[{]attribute-value[}]
attribute-keyword ::= identifier
attribute-value ::= character-string
```

To put it simple connection string is just list of the key=value entries separated by semicolon. You can find connection string samples below.
[block:api-header]
{
  "type": "basic",
  "title": "Supported arguments"
}
[/block]
Apache Ignite ODBC driver supports and uses following connection string/DSN
arguments:
[block:parameters]
{
  "data": {
    "h-0": "Attribute keyword",
    "h-1": "Description",
    "h-2": "Default value",
    "0-0": "SERVER",
    "1-0": "PORT",
    "2-0": "CACHE",
    "2-1": "Cache name. If it is not defined than default cache will be used. Note that the cache name is case sensitive.",
    "1-1": "**Deprecated. Please, use ADDRESS attribute instead.**\nPort on which OdbcProcessor of the node is listening.",
    "0-1": "**Deprecated. Please, use ADDRESS attribute instead.**\nAddress of the node to connect to.",
    "0-2": "",
    "1-2": "10800",
    "3-0": "ADDRESS",
    "3-1": "Address of the remote node to connect to. The format: <host>[:<port>]. For example: localhost, example.com:12345, 127.0.0.1, 192.168.3.80:5893.",
    "5-0": "PROTOCOL_VERSION",
    "4-0": "DSN",
    "4-1": "Use to connect using specified DSN.",
    "5-1": "Used to specify ODBC protocol version to use. Currently, there are only two versions: 1.6.0 and 1.8.0. You should use 1.6.0 protocol version to connect to nodes with Ignite version < 1.8.0.",
    "5-2": "1.8.0",
    "6-0": "PAGE_SIZE",
    "6-1": "Number of rows returned in response to the one request to the data source. Default value should be fine for the most of use cases. Setting low value can result in slow data fetching while setting high value can result in additional memory usage by the driver and additional delay when the next page is being retrieved.",
    "6-2": "1024"
  },
  "cols": 3,
  "rows": 7
}
[/block]
All parameter names are case-insensitive so `SERVER`, `Server` and `server` all are
valid parameter names and refer to the same parameter.
[block:api-header]
{
  "type": "basic",
  "title": "Connection string samples"
}
[/block]
You can find samples of the connection string below. These strings can be used with `SQLDriverConnect` ODBC call to establish connection with an Apache Ignite node.
[block:code]
{
  "codes": [
    {
      "code": "DRIVER={Apache Ignite};ADDRESS=localhost:10800;CACHE=MyCache",
      "language": "text",
      "name": "Specific cache"
    },
    {
      "code": "DRIVER={Apache Ignite};ADDRESS=localhost:10800",
      "language": "text",
      "name": "Default cache"
    },
    {
      "code": "DSN=MyIgniteDSN",
      "language": "text",
      "name": "DSN"
    },
    {
      "code": "DRIVER={Apache Ignite};ADDRESS=example.com:12901;CACHE=SomeCache;PROTOCOL_VERSION=1.6.0",
      "language": "text",
      "name": "Legacy Node"
    },
    {
      "code": "DRIVER={Apache Ignite};ADDRESS=example.com:12901;CACHE=MyCache;PAGE_SIZE=4096",
      "language": "text",
      "name": "Custom page size"
    }
  ]
}
[/block]