---
title: "Java"
excerpt: ""
---
To connect to Ignite using Java client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

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
      "code": "MemcachedClient client = null;\n\ntry {\n    client = new MemcachedClient(new BinaryConnectionFactory(),\n            AddrUtil.getAddresses(\"localhost:11211\"));\n} catch (IOException e) {\n    e.printStackTrace();\n}\n\nclient.set(\"key\", 0, \"val\");\n\nSystem.out.println(\"Value for 'key': \" + c.get(\"key\"));",
      "language": "java"
    }
  ]
}
[/block]