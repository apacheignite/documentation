Topology validator checks whether the new topology is valid for specific cache at each topology change.

Topology is always valid in case no topology validator used.
In case topology is valid for specific cache all operations on this cache are allowed.
In case topology is not valid for specific cache all update operations on this cache are restricted:
   - CacheException will be thrown at update operations (put, remove, etc) attempt.
   - IgniteException will be thrown at transaction commit attempt.
[block:code]
{
  "codes": [
    {
      "code": "...\nfor (CacheConfiguration cCfg : iCfg.getCacheConfiguration()) {\n    if (cCfg.getName() != null) {\n        if (cCfg.getName().equals(CACHE_NAME_1))\n            cCfg.setTopologyValidator(new TopologyValidator() {\n                @Override public boolean validate(Collection<ClusterNode> nodes) {\n                    return nodes.size() == 2;\n                }\n                    });\n        else if (cCfg.getName().equals(CACHE_NAME_2))\n            cCfg.setTopologyValidator(new TopologyValidator() {\n                @Override public boolean validate(Collection<ClusterNode> nodes) {\n                    return nodes.size() >= 2;\n                }\n            });\n    }\n}\n...",
      "language": "java",
      "name": "Setup example"
    }
  ]
}
[/block]
In this case cache with name `CACHE_NAME_1` will allow update operations in case cluster contains exactly 2 nodes, but cache with name `CACHE_NAME_2` will allow update operations in case cluster contain at least 2 nodes.