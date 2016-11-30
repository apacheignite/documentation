* [Connection String Format](#connection-string-format)
* [Supported Arguments](#supported-arguments)
* [Connection String Example](#connection-string-example)
* [Configuring DSN](#configuring-dsn)
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
Apache Ignite ODBC driver supports and uses several connection string/DSN arguments. All parameter names are case-insensitive so `ADDRESS`, `Address` and `address` all are valid parameter names and refer to the same parameter. If argument is not specified, the default value is used. The exception of this rule is `ADDRESS` attribute. If it is not specified, `SERVER` and `PORT` attributes are used instead.
[block:parameters]
{
  "data": {
    "h-0": "Attribute keyword",
    "h-1": "Description",
    "h-2": "Default value",
    "1-0": "SERVER",
    "2-0": "PORT",
    "3-0": "CACHE",
    "3-1": "Cache name. If it is not defined than default cache will be used. Note that the cache name is case sensitive.",
    "2-1": "Port on which `OdbcProcessor` of the node is listening.\n\nThis argument value is ignored if `ADDRESS` argument is specified.",
    "1-1": "Address of the node to connect to.\n\nThis argument value is ignored if `ADDRESS` argument is specified.",
    "1-2": "",
    "2-2": "10800",
    "6-0": "DISTRIBUTED_JOINS",
    "4-0": "DSN",
    "4-1": "DSN name to connect to.",
    "6-1": "Enables [non-collocated distributed joins feature](http://apacheignite.readme.io/docs/sql-queries#distributed-joins) for all the queries that will be executed over an ODBC connection.",
    "6-2": "false",
    "0-0": "ADDRESS",
    "0-1": "Address of the remote node to connect to. The format: <host>[:<port>]. For example: localhost, example.com:12345, 127.0.0.1, 192.168.3.80:5893.\n\nIf this attribute is specified then `SERVER` and `PORT` arguments are ignored.",
    "5-0": "PAGE_SIZE",
    "5-1": "Number of rows returned in response to a fetching request to the data source. Default value should be fine in most cases. Setting low value can result in slow data fetching while setting high value can result in additional memory usage by the driver and additional delay when the next page is being retrieved.",
    "5-2": "1024",
    "8-0": "PROTOCOL_VERSION",
    "8-1": "Used to specify ODBC protocol version to use. Currently, there are only two versions: 1.6.0 and 1.8.0. You should use 1.6.0 protocol version to connect to nodes with Ignite version < 1.8.0.",
    "8-2": "1.8.0",
    "7-0": "ENFORCE_JOIN_ORDER",
    "7-1": "Enforces a join order of tables in SQL queries. If set to `true` the query optimizer will not reorder tables in the join.",
    "7-2": "false"
  },
  "cols": 3,
  "rows": 9
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Connection String Samples"
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
      "name": "Legacy node"
    },
    {
      "code": "DRIVER={Apache Ignite};ADDRESS=example.com:12901;CACHE=MyCache;PAGE_SIZE=4096",
      "language": "text",
      "name": "Custom page size"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring DSN"
}
[/block]
The same arguments can be used if you prefer to use [DSN](https://en.wikipedia.org/wiki/Data_source_name) (Data Source Name) for connection purposes. 

To configure DSN on Windows, you should use a system tool called `odbcad32` which is an ODBC Data Source Administrator. To launch this tool, go to `Control panel`->`Administrative Tools`->`Data Sources (ODBC)`. Once the ODBC Data Source Administrator is launched, select `Add...`->`Apache Ignite` and configure your DSN in the desired way.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/6255aff-DSN_configuration.png",
        "DSN_configuration.png",
        607,
        427,
        "#efefee"
      ]
    }
  ]
}
[/block]
To do the same on Linux, you have to locate the `odbc.ini` file. The file location varies among Linux distributions and depends on a specific Driver Manager used by a Linux distribution. As an example, if you are using `unixODBC` then you can run the following command that will print out system wide ODBC related details:
[block:code]
{
  "codes": [
    {
      "code": "odbcinst -j",
      "language": "shell"
    }
  ]
}
[/block]
A path to the file of interest will be shown in between `SYSTEM DATA SOURCES` and 
`USER DATA SOURCES` properties.

Once the `odbc.ini` file has been found, open it with any editor of your choice and add DSN section to it as shown below:

[block:code]
{
  "codes": [
    {
      "code": "[DSN Name]\ndescription=<Insert your description here>\ndriver=Apache Ignite\n<Other arguments here...>",
      "language": "text",
      "name": "odbc.ini"
    }
  ]
}
[/block]