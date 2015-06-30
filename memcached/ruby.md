---
title: "Ruby"
excerpt: ""
---
To connect to Ignite using Ruby client for Memcached, you need to [download Ignite](https://ignite.incubator.apache.org/download.html) and - 

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
      "code": "require 'dalli'\n\noptions = { :namespace => \"app_v1\", :compress => true }\n\nclient = Dalli::Client.new('localhost:11211', options)\n\nclient.set('key', 'value')\n\nvalue = client.get('key')",
      "language": "ruby"
    }
  ]
}
[/block]