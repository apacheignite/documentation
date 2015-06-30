---
title: "Python"
excerpt: ""
---
To connect to Ignite using Python client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

1\. Start Ignite cluster with cache configured. For example:
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
      "code": "import pylibmc\n\nclient = memcache.Client([\"127.0.0.1:11211\", binary=True])\n\nclient.set(\"key\", \"val\")\n\nprint \"Value for 'key': %s\" % \n\nclient.get(\"key\")",
      "language": "python"
    }
  ]
}
[/block]