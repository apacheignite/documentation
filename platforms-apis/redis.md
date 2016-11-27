Ignite is partially [Redis](http://redis.io/) compliant which enables users to store and retrieve distributed data from Ignite cache using any Redis compatible client.

Starting from version 1.8.0 the following commands are supported by Ignite client:
- [ECHO](http://redis.io/commands/echo)
- [PING](http://redis.io/commands/ping)
- [QUIT](http://redis.io/commands/quit)
- [GET](http://redis.io/commands/get)
- [MGET](http://redis.io/commands/mget) (*limitation: not null values returned for non-existing keys*)
- [SET](http://redis.io/commands/set) (*limitation: without key expiration*)
- [MSET](http://redis.io/commands/mset)
- [INCR](http://redis.io/commands/incr)
- [DECR](http://redis.io/commands/decr)
- [INCRBY](http://redis.io/commands/incrby)
- [DECRBY](http://redis.io/commands/decrby)
- [APPEND](http://redis.io/commands/append)
- [STRLEN](http://redis.io/commands/strlen)
- [GETSET](http://redis.io/commands/getset)
- [SETRANGE](http://redis.io/commands/setrange)
- [GETRANGE](http://redis.io/commands/getrange)
- [DEL](http://redis.io/commands/del) (*limitation: number of submitted keys returned*)
- [EXISTS](http://redis.io/commands/exists)
- [DBSIZE](http://redis.io/commands/dbsize)

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
  "body": "In Redis, databases are identified by an integer index, not by a database name. Therefore, currently you cannot switch between caches by names and only entries from <default> cache are handled."
}
[/block]
You can connect to Ignite using your favorite [Redis client](http://redis.io/clients). See basic examples for some languages below
- [Java](#java)
- [Python](#python)
- [PHP](#php)
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

[block:api-header]
{
  "type": "basic",
  "title": "PHP"
}
[/block]
To connect to Ignite using a PHP client for Redis, you need to have an Ignite cluster/node configured, if necessary, as shown [above](doc:redis) and running.

To connect to Ignite running on port `6379` with, for instance, [predis](https://github.com/nrk/predis)
[block:code]
{
  "codes": [
    {
      "code": "// Load the library.\nrequire 'predis/autoload.php';\nPredis\\Autoloader::register();\n\n// Connect.\ntry {\n    $redis = new Predis\\Client();\n\n    echo \">>> Successfully connected to Redis. \\n\";\n\n    // Put entry to cache.\n    if ($redis->set('k1', '1'))\n        echo \">>> Successfully put entry in cache. \\n\";\n\n    // Check entry value.\n    echo(\">>> Value for 'k1': \" . $redis->get('k1') . \"\\n\");\n}\ncatch (Exception $e) {\n    echo \"Couldn't connected to Redis\";\n    echo $e->getMessage();\n}",
      "language": "php"
    }
  ]
}
[/block]