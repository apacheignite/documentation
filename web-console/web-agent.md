[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite Web Agent is a java standalone application that allow to connect Ignite Grid to Ignite Web Console.
Ignite Web Agent communicates with grid nodes via REST interface and connects to Ignite Web Console via web-socket.

Two main functions of Ignite Web Agent:
* Proxy between Ignite Web Console and Ignite Grid to execute SQL statements and collect metrics for monitoring.
   You may need to specify URI for connect to Ignite REST server via "-n" option.

* Proxy between Ignite Web Console and user RDBMS to collect database metadata for later CacheTypeMetadata configuration.
   You may need to copy JDBC driver into "./jdbc-drivers" subfolder or specify path via "-d" option.