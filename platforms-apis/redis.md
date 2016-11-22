Ignite is partially [Redis](http://redis.io/) compliant which enables users to store and retrieve distributed data from Ignite cache using any Redis compatible client.

In version 1.8.0 only the following commands are supported
- ECHO
- PING
- QUIT
- GET
- MGET (*limitation: not null values returned for non-existing keys*)
- SET (*limitation: without key expiration*)
- MSET
- INCR
- DECR
- INCRBY
- DECRBY
- APPEND
- STRLEN
- GETSET
- SETRANGE
- GETRANGE
- DEL (*limitation: number of submitted keys returned*)
- EXISTS
- DBSIZE

Cluster nodes accepts Redis requests listening on a particular socket. By default each Ignite node is listening for incoming requests on `[host]:11211`. You can override the host and port using `ConnectorConfiguration`
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  \t...\n\t<property name=\"connectorConfiguration\">\n\t    <bean class=\"org.apache.ignite.configuration.ConnectorConfiguration\">\n\t\t<property name=\"host\" value=\"localhost\"/>\n\t\t<property name=\"port\" value=\"6379\"/>\n\t    </bean>\n\t</property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "In Redis, databases are identified by an integer index, not by a database name. Therefore, currently you cannot switch between caches by names and only entries from null cache are handled."
}
[/block]
You can connect to Ignite using your favorite [Redis client](http://redis.io/clients). See basic examples for some languages below
- [Java](#java)
- [Python](#python)
[block:api-header]
{
  "type": "basic",
  "title": "Java"
}
[/block]
To connect to Ignite using a Java client for Redis, you need to have an Ignite cluster/node configured, if necessary, as shown [above](doc:redis) and running.

To connect to Ignite running on port `6379` with, for instance, [Jedis](https://github.com/xetorthio/jedis)
[block:code]
{
  "codes": [
    {
      "code": "JedisPoolConfig jedisPoolCfg = new JedisPoolConfig();\n\n// your pool configurations.\n\nJedisPool pool = new JedisPool(jedisPoolCfg, \"localhost\", 6379, 10000);\n\ntry (Jedis jedis = pool.getResource()) {\n    jedis.set(\"key1\", \"1\");\n    System.out.println(\"Value for 'key1': \" + jedis.get(\"key1\"));\n}\n\npool.destroy();",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Python"
}
[/block]
To connect to Ignite using a Python client for Redis, you need to have an Ignite cluster/node configured, if necessary, as shown [above](doc:redis) and running.

To connect to Ignite running on port `6379` with, for instance, [redis-py](https://github.com/andymccurdy/redis-py)
[block:code]
{
  "codes": [
    {
      "code": ">>> import redis\n>>> r = redis.StrictRedis(host='localhost', port=6379, db=0)\n>>> r.set('k1', 1)\nTrue\n>>> r.get('k1')\n'1'",
      "language": "python"
    }
  ]
}
[/block]