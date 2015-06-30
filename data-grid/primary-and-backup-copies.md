---
title: "Primary & Backup Copies"
excerpt: ""
---
In `PARTITIONED` mode, nodes to which the keys are assigned to are called primary nodes for those keys. You can also optionally configure any number of backup nodes for cached data. If the number of backups is greater than 0, then Ignite will automatically assign backup nodes for each individual key. For example if the number of backups is 1, then every key cached in the data grid will have 2 copies, 1 primary and 1 backup.
[block:callout]
{
  "type": "info",
  "body": "By default, backups are turned off for better performance."
}
[/block]
Backups can be configured by setting `backups()` property of `CacheConfiguration`, like so:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  \t...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n           \t<!-- Set a cache name. -->\n           \t<property name=\"name\" value=\"cacheName\"/>\n          \n          \t<!-- Set cache mode. -->\n    \t\t\t\t<property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          \t\n          \t<!-- Number of backup nodes. -->\n    \t\t\t\t<property name=\"backups\" value=\"1\"/>\n    \t\t\t\t... \n        </bean\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\ncacheCfg.setCacheMode(CacheMode.PARTITIONED);\n\ncacheCfg.setBackups(1);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Synchronous and Asynchronous Backups"
}
[/block]
`CacheWriteSynchronizationMode` enum can be used to configure synchronous or asynchronous update of primary and backup copies. Write synchronization mode tells Ignite whether the client node should wait for responses from remote nodes, before completing a write or commit. 

Write synchronization mode can be set in one of following 3 modes:
[block:parameters]
{
  "data": {
    "0-0": "`FULL_SYNC`",
    "1-0": "`FULL_ASYNC`",
    "2-0": "`PRIMARY_SYNC`",
    "0-1": "Client node will wait for write or commit to complete on all participating remote nodes (primary and backup).",
    "1-1": "This is the default value. In this mode, client node does not wait for responses from participating nodes, in which case remote nodes may get their state updated slightly after any of the cache write methods complete or after `Transaction.commit()` method completes.",
    "2-1": "Client node will wait for write or commit to complete on primary node, but will not wait for backups to be updated."
  },
  "cols": 2,
  "rows": 3
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Note that regardless of write synchronization mode, cache data will always remain fully consistent across all participating nodes.",
  "title": "Cache Data Consistency"
}
[/block]
Write synchronization mode may be configured by setting `writeSynchronizationMode` property of `CacheConfiguration`, like so:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  \t...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n           \t<!-- Set a cache name. -->\n           \t<property name=\"name\" value=\"cacheName\"/>\n          \n          \t<!-- Set write synchronization mode. -->\n    \t\t\t\t<property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>      \t\n    \t\t\t\t... \n        </bean\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\ncacheCfg.setWriteSynchronizationMode(CacheWriteSynchronizationMode.FULL_SYNC);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]