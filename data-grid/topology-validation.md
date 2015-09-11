Topology validator is used to verify that cluster topology is valid for further cache operations. 

The topology validator is invoked every time the cluster topology changes (either a new node joined or an existing node failed or left). If topology validator is not configured, then the cluster topology is always considered to be valid.

Whenever the `validate(Collection)` method returns true, then the topology is considered valid for a certain cache and all operations on this cache will be allowed to proceed. Otherwise, all update operations on the cache are restricted with the following exceptions:
- `CacheException` will be thrown for all update operations (put, remove, etc) attempt.
- `IgniteException` will be thrown for the transaction commit attempt.

After returning false and declaring the topology not valid, the topology validator can return to normal state whenever the next topology change happens.

Example
[block:code]
{
  "codes": [
    {
      "code": "...\nfor (CacheConfiguration cCfg : iCfg.getCacheConfiguration()) {\n    if (cCfg.getName() != null) {\n        if (cCfg.getName().equals(CACHE_NAME_1))\n            cCfg.setTopologyValidator(new TopologyValidator() {\n                @Override public boolean validate(Collection<ClusterNode> nodes) {\n                    return nodes.size() == 2;\n                }\n            });\n        else if (cCfg.getName().equals(CACHE_NAME_2))\n            cCfg.setTopologyValidator(new TopologyValidator() {\n                @Override public boolean validate(Collection<ClusterNode> nodes) {\n                    return nodes.size() >= 2;\n                }\n            });\n    }\n}\n...",
      "language": "java",
      "name": "Setup example"
    }
  ]
}
[/block]
In this example update operations will be allowed to cache with name
- `CACHE_NAME_1` in case cluster contains exactly 2 nodes
- `CACHE_NAME_2` in case cluster contain at least 2 nodes.

Configuration
The topology validator can be configured either from code or XML via `CacheConfiguration.setTopologyValidator(TopologyValidator)` method.