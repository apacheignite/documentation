## On this page
* [Connection String Format](#connection-string-format)
* [Supported Arguments](#supported-arguments)
* [Connection String Example](#connection-string-samples)
[block:api-header]
{
  "type": "basic",
  "title": "Connection String Format"
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
  "title": "Supported Arguments"
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
    "1-1": "Port on which OdbcProcessor of the node is listening.",
    "0-1": "Address of the node to connect to.",
    "0-2": "lacalhost",
    "1-2": "10800"
  },
  "cols": 3,
  "rows": 3
}
[/block]
All parameter names are case-insensitive so `SERVER`, `Server` and `server` all are
valid parameter names and refer to the same parameter.
[block:api-header]
{
  "type": "basic",
  "title": "Connection String Example"
}
[/block]
You can find examples of the connection string below. These strings can be used with `SQLDriverConnect` ODBC call to establish connection with an Apache Ignite node.
[block:code]
{
  "codes": [
    {
      "code": "DRIVER={Apache Ignite};SERVER=localhost;PORT=10800;CACHE=MyCache",
      "language": "text",
      "name": "Specific cache"
    },
    {
      "code": "DRIVER={Apache Ignite};SERVER=localhost;PORT=10800",
      "language": "text",
      "name": "Default cache"
    }
  ]
}
[/block]