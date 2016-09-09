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
[block:callout]
{
  "type": "info",
  "body": "* In order to communicate with web agent Ignite node should be started with REST server (move `ignite-rest-http` folder from `lib/optional/` to `lib/`).\n* Configure web agent `serverURI` property by Ignite node REST server URI.",
  "title": "Requirements"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
###Configuration file
By default Ignite Web Agent try to load configuration settings from `default.properties` file.
This file must have simple line-oriented format as described [here](#http://docs.oracle.com/javase/7/docs/api/java/util/Properties.html).

Available entries names: `tokens`, `server-uri`, `node-uri`, `driver-folder`
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
###Command line options
To get help execute `ignite-web-agent.{sh|bat} -h` or  `ignite-web-agent.{sh|bat} --help` in terminal.

Available options with they descriptions:
* `-h`, `--help` - Print this help message
* `-c`, `--config` - Path to configuration file
* `-d`, `--driver-folder` - Path to folder with JDBC drivers, default value: ./jdbc-drivers
* `-n`, `--node-uri` - URI for connect to Ignite REST server, default value: `http://localhost:8080`
* `-s`, `--server-uri` - URI for connect to Ignite Web Console, default value: `http://localhost:3001`
* `-t`, `--tokens` - User's security tokens