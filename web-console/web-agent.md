[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite Web Agent is a java standalone application that allow to connect Ignite Grid to Ignite Web Console.
Ignite Web Agent communicates with grid nodes via REST interface and connects to Ignite Web Console via web-socket.

Two main functions of Ignite Web Agent:
* Proxy between Ignite Web Console and Ignite Grid to execute SQL statements.
* Proxy between Ignite Web Console and user RDBMS to collect database metadata for later CacheTypeMetadata configuration.
[block:api-header]
{
  "type": "basic",
  "title": "Usage"
}
[/block]
Ignite Web Agent zip ships with `ignite-web-agent.{sh|bat}` script that used for agent start.
To get help execute `ignite-web-agent.{sh|bat} -h` or  `ignite-web-agent.{sh|bat} --help` in terminal.
[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
###Configuration file
By default Ignite Web Agent try to load configuration setting from `default.properties`
This file must have simple line-oriented format as described here: http://docs.oracle.com/javase/7/docs/api/java/util/Properties.html#load(java.io.Reader)

Available entries names: `tokens`, `server-uri`, `node-uri`, `driver-folder`

Example configuration file:  
[block:code]
{
  "codes": [
    {
      "code": "tokens=1a2b3c4d5f,2j1s134d12\nserverURI=https://console.example.com:3001",
      "language": "text",
      "name": "default.properties"
    }
  ]
}
[/block]