---
title: "PHP"
excerpt: ""
---
To connect to Ignite using PHP client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

1\. Start Ignite cluster with cache configured. For example :
[block:code]
{
  "codes": [
    {
      "code": "bin/ignite.sh examples/config/example-cache.xml",
      "language": "text"
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
      "language": "java"
    }
  ]
}
[/block]