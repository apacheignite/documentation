Ignite is [Memcached](http://memcached.org/) compliant which enables users to store and retrieve distributed data from Ignite cache using any Memcached compatible client.
[block:callout]
{
  "type": "info",
  "body": "Currently, Ignite supports only binary protocol for Memcached."
}
[/block]
You can connect to Ignite using a Memcached client in one of the following languages:

 * [PHP](#php)
 * [Java](#java)
 * [Python](#python)
 * [Ruby](#ruby)
[block:api-header]
{
  "type": "basic",
  "title": "PHP"
}
[/block]
To connect to Ignite using PHP client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

1\. Start Ignite cluster with cache configured. For example :
[block:code]
{
  "codes": [
    {
      "code": "bin/ignite.sh examples/config/example-cache.xml",
      "language": "shell"
    }
  ]
}
[/block]
2\. Connect to Ignite using  Memcached client, via binary protocol.
[block:code]
{
  "codes": [
    {
      "code": "// Create client instance.\n$client = new Memcached();\n\n// Set localhost and port (set to correct values).\n$client->addServer(\"localhost\", 11211);\n\n// Force client to use binary protocol.\n$client->setOption(Memcached::OPT_BINARY_PROTOCOL, true);\n\n// Put entry to cache.\nif ($client->add(\"key\", \"val\"))\n    echo \"Successfully put entry in cache.\\n\";\n\n// Check entry value.\necho(\"Value for 'key': \" . $client->get(\"key\") . \"\\n\");",
      "language": "php"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Java"
}
[/block]
To connect to Ignite using Java client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

1\. Start Ignite cluster with cache configured. For example:
[block:code]
{
  "codes": [
    {
      "code": "bin/ignite.sh examples/config/example-cache.xml",
      "language": "shell"
    }
  ]
}
[/block]
2\. Connect to Ignite using  Memcached client, via binary protocol.
[block:code]
{
  "codes": [
    {
      "code": "MemcachedClient client = null;\n\ntry {\n    client = new MemcachedClient(new BinaryConnectionFactory(),\n            AddrUtil.getAddresses(\"localhost:11211\"));\n} catch (IOException e) {\n    e.printStackTrace();\n}\n\nclient.set(\"key\", 0, \"val\");\n\nSystem.out.println(\"Value for 'key': \" + client.get(\"key\"));",
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
To connect to Ignite using Python client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

1\. Start Ignite cluster with cache configured. For example:
[block:code]
{
  "codes": [
    {
      "code": "bin/ignite.sh examples/config/example-cache.xml",
      "language": "shell"
    }
  ]
}
[/block]
2\. Connect to Ignite using  Memcached client, via binary protocol.
[block:code]
{
  "codes": [
    {
      "code": "import pylibmc\n\nclient = pylibmc.Client ([\"127.0.0.1:11211\"], binary=True)\n\nclient.set(\"key\", \"val\")\n\nprint \"Value for 'key': %s\"%client.get(\"key\")",
      "language": "python"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Ruby"
}
[/block]
To connect to Ignite using Ruby client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

1\. Start Ignite cluster with cache configured. For example:
[block:code]
{
  "codes": [
    {
      "code": "bin/ignite.sh examples/config/example-cache.xml",
      "language": "shell"
    }
  ]
}
[/block]
2\. Connect to Ignite using  Memcached client, via binary protocol.
[block:code]
{
  "codes": [
    {
      "code": "require 'dalli'\n\noptions = { :namespace => \"app_v1\", :compress => true }\n\nclient = Dalli::Client.new('localhost:11211', options)\n\nclient.set('key', 'value')\n\nvalue = client.get('key')",
      "language": "ruby"
    }
  ]
}
[/block]