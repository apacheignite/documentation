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